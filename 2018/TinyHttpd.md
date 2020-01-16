## TinyHttpd源码分析

> 原创 涂飞平  `2018-04-20`

TinyHttpd是一个C编写的极小HTTP服务器，代码量（包括注释）不到500行，但可以基本说明HTTP服务器的工作原理，从中也能知道如何进行Linux下的C编程（这是我看这个代码的主要原因）。
在Linux下面，我采用Eclipse CPP作为C/C++的IDE，设置好相应的环境变量，以便IDE可以查找链接相应的头文件和库。

<p style="text-align: center;"><img src="https://raw.githubusercontent.com/tufeiping/tufeiping.github.io/master/assets/tinyhttpd.png" alt="tinyhttpd.png"></p>

源码分析过程如下:

~~~java
	// 函数声明
	void accept_request(int); //接受请求
	void bad_request(int); //错误请求
	void cat(int, FILE *);
	void cannot_execute(int); //运行失败
	void error_die(const char *); //异常
	void execute_cgi(int, const char *, const char *, const char *); //执行cgi程序
	int get_line(int, char *, int); // 读取每行
	void headers(int, const char *); //获取头部信息
	void not_found(int); // 404处理
	void serve_file(int, const char *); //静态文件请求
	int startup(u_short *); //启动服务
	void unimplemented(int); //未实现
~~~

下面先看启动函数-- **main**

~~~java
int main(void) {
	int server_sock = -1;
	u_short port = 0;
	int client_sock = -1;
	struct sockaddr_in client_name;
	int client_name_len = sizeof(client_name);
	pthread_t newthread;
	//初始化server socket port->返回随机绑定的端口，当然，port可以指定端口
	server_sock = startup(&port); 
	printf("httpd running on port %d\n", port);
	//轮询，接受客户端请求并完成请求
	while (1) {
		client_sock = accept(server_sock, (struct sockaddr *) &client_name, &client_name_len);
		if (client_sock == -1)
			error_die("accept");
		//对于每个客户端请求，使用一个线程来处理，线程函数为accept_request
		if (pthread_create(&newthread, NULL, accept_request, client_sock) != 0)
			perror("pthread_create");
	}
	//关闭server socket
	close(server_sock);

	return (0);
}
~~~

对于server部分，都是socket编程的标准套路。http server对于客户请求的响应逻辑在accept_request函数中。

~~~java
void accept_request(int client) {
	char buf[1024];
	int numchars;
	char method[255];
	char url[255];
	char path[512];
	size_t i, j;
	struct stat st;
	int cgi = 0; /* becomes true if server decides this is a CGI
	* program */
	char *query_string = NULL;

	//从客户socket中按行读取到buf中
	numchars = get_line(client, buf, sizeof(buf)); 
	i = 0;
	j = 0;

	/** 
	HTTP头第一行格式如下
	METHOD QUERY_URL
	**/
	//读取mthod
	while (!ISspace(buf[j]) && (i < sizeof(method) - 1)) {
		method[i] = buf[j];
		i++;
		j++;
	}
	method[i] = '\0';
	//方法仅支持GET和POST两种，其他返回“未实现”信息
	if (strcasecmp(method, "GET") && strcasecmp(method, "POST")) {
		unimplemented(client);
		return;
	}

	//如果是POST请求，说明有参数存在，交给CGI处理
	if (strcasecmp(method, "POST") == 0)
		cgi = 1;

	i = 0;
	while (ISspace(buf[j]) && (j < sizeof(buf)))
		j++;
	//解析请求的URL
	while (!ISspace(buf[j]) && (i < sizeof(url) - 1) && (j < sizeof(buf))) {
		url[i] = buf[j];
		i++;
		j++;
	}
	url[i] = '\0';

	if (strcasecmp(method, "GET") == 0) {
		query_string = url;
		while ((*query_string != '?') && (*query_string != '\0'))
			query_string++;
		//如果GET方法的请求串中带有?，说明有参数存在，交给CGI处理
		if (*query_string == '?') {
			cgi = 1;
			// 加入c字符串终结符\0，url就是?前部分
			*query_string = '\0';
			//query_string指向?后面的参数串
			query_string++; 
		}
	}

	//先将url与htdocs合并，查找是否有相关文件
	sprintf(path, "htdocs%s", url);
	//如果以/结尾，自动定位路径下的index.html
	if (path[strlen(path) - 1] == '/')
		strcat(path, "index.html");
	if (stat(path, &st) == -1) { //如果文件不存在
		while ((numchars > 0) && strcmp("\n", buf)) //将剩余的内容读取并丢弃
			numchars = get_line(client, buf, sizeof(buf));
		not_found(client); // 显示404 not found
	} else {
		if ((st.st_mode & S_IFMT) == S_IFDIR)
			strcat(path, "/index.html"); //如果是目录，指向目录下的index.html文件
		if ((st.st_mode & S_IXUSR) || (st.st_mode & S_IXGRP) || (st.st_mode & S_IXOTH)) //如果文件是可执行的
			cgi = 1;
		if (!cgi)
			serve_file(client, path); //将文件读出并送到客户端
		else
			execute_cgi(client, path, method, query_string); //交给cgi处理
	}

	close(client); // 处理完毕，关闭客户端连接
}

// 文件服务，将文件读取并传到客户端
void serve_file(int client, const char *filename) {
	FILE *resource = NULL;
	int numchars = 1;
	char buf[1024];

	buf[0] = 'A';
	buf[1] = '\0';
	while ((numchars > 0) && strcmp("\n", buf)) //读取丢弃剩余头部内容
		numchars = get_line(client, buf, sizeof(buf));
	//按文本方式读取文件
	resource = fopen(filename, "r");
	if (resource == NULL)
		not_found(client);
	else {
		headers(client, filename); //组装http响应头
		cat(client, resource); //返回文件内容
	}
	fclose(resource); //关闭文件
}

//组装http响应头
void headers(int client, const char *filename) {
	char buf[1024];
	(void) filename; /* could use filename to determine file type */
	//将内容拷贝到buf中并发出，返回状态码 200
	strcpy(buf, "HTTP/1.0 200 OK\r\n");
	send(client, buf, strlen(buf), 0);
	//返回server标识
	strcpy(buf, SERVER_STRING); 
	//#define SERVER_STRING "Server: jdbhttpd/0.1.0\r\n"
	send(client, buf, strlen(buf), 0);
	//返回mime-type信息
	sprintf(buf, "Content-Type: text/html\r\n");
	send(client, buf, strlen(buf), 0);
	strcpy(buf, "\r\n");
	//发生分割符\r\n，准备发送实际内容
	send(client, buf, strlen(buf), 0);
}

//发送文件内容
void cat(int client, FILE *resource) {
	char buf[1024];
	//按行读取文件，并按行发送到客户端
	fgets(buf, sizeof(buf), resource);
	while (!feof(resource)) {
		send(client, buf, strlen(buf), 0);
		fgets(buf, sizeof(buf), resource);
	}
}
~~~

以上的内容是静态文件处理方式。下面分析cgi的处理方式，从execute_cgi开始分析。
个人认为execute_cgi是最具价值的代码块，从这部分代码可以理解linux下面如何创建进程，如何共享环境变量，如何使用管道来进行进程间通信。

~~~java
void execute_cgi(int client, const char *path, const char *method, const char *query_string) {
	char buf[1024];
	int cgi_output[2];
	int cgi_input[2];
	pid_t pid;
	int status;
	int i;
	char c;
	int numchars = 1;
	int content_length = -1;

	buf[0] = 'A';
	buf[1] = '\0';
	if (strcasecmp(method, "GET") == 0) //GET方法
		while ((numchars > 0) && strcmp("\n", buf)) //忽略剩余的头部内容
			numchars = get_line(client, buf, sizeof(buf));
	else //POST方法
	{
		numchars = get_line(client, buf, sizeof(buf));
		while ((numchars > 0) && strcmp("\n", buf)) {
			buf[15] = '\0';
			if (strcasecmp(buf, "Content-Length:") == 0)
				content_length = atoi(&(buf[16]));
			numchars = get_line(client, buf, sizeof(buf));
		}
		if (content_length == -1) { 
			//如果POST没有Content-Length域，则认为是错误请求，因为后面会通过这个
			//域的大小读取内容并交给cgi处理
			bad_request(client);
			return;
		}
	}
	//先发送状态码 200
	sprintf(buf, "HTTP/1.0 200 OK\r\n");
	send(client, buf, strlen(buf), 0);
	//创建输出管道
	if (pipe(cgi_output) < 0) {
		cannot_execute(client); //如果管道创建失败，返回错误信息
		//个人感觉，状态码应该在这里发送，并返回50x状态码说明内部错误
		return;
	}
	//创建输入管道
	if (pipe(cgi_input) < 0) {
		cannot_execute(client);
		return;
	}
	//fork一个进程，如果失败，同样处理
	if ((pid = fork()) < 0) {
		cannot_execute(client);
		return;
	}
	if (pid == 0) /* child: CGI script */
	{
		//pid为0,说明fork分支路径在执行，下面是子进程处理逻辑
		char meth_env[255];
		char query_env[255];
		char length_env[255];

		dup2(cgi_output[1], 1);//标准输出重定向为cgi_output[1]
		dup2(cgi_input[0], 0); //标准输入重定向为cgi_input[0]
		close(cgi_output[0]); 
		close(cgi_input[1]);
		//即使在fork子进程中，所有的变量还是可以读取的
		sprintf(meth_env, "REQUEST_METHOD=%s", method);
		//将REQUEST_METHOD加入到env中
		putenv(meth_env);
		if (strcasecmp(method, "GET") == 0) { 
		//将请求串也加入到env中 QUERY_STRING
			sprintf(query_env, "QUERY_STRING=%s", query_string);
			putenv(query_env);
		} else { /* POST */
			sprintf(length_env, "CONTENT_LENGTH=%d", content_length);
			putenv(length_env);
		}
		//执行文件，这之后，进程完全分离了，不会再返回回来了
		execl(path, path, NULL);
		//如果execl执行失败，退出
		exit(0);
	} else { /* parent */
		//pid不为0,说明还在自身的进程路径下执行
		close(cgi_output[1]);
		close(cgi_input[0]);
		//通过管道与子进程通信
		if (strcasecmp(method, "POST") == 0) //POST将请求内容发给管道
			for (i = 0; i < content_length; i++) {
				recv(client, &c, 1, 0);
				write(cgi_input[1], &c, 1);
			}
			//返回子进程的输出内容
		while (read(cgi_output[0], &c, 1) > 0)
			send(client, &c, 1, 0);

		close(cgi_output[0]);
		close(cgi_input[1]);
		//等待子进程退出
		waitpid(pid, &status, 0);
	}
}

~~~

分析完核心函数，cannot_execute，bad_request，not_found，unimplemented等函数均是显示一些错误提示信息，这里仅贴出unimplemented函数实现。

~~~java
void unimplemented(int client) {
	char buf[1024];

	sprintf(buf, "HTTP/1.0 501 Method Not Implemented\r\n");
	send(client, buf, strlen(buf), 0);
	sprintf(buf, SERVER_STRING);
	send(client, buf, strlen(buf), 0);
	sprintf(buf, "Content-Type: text/html\r\n");
	send(client, buf, strlen(buf), 0);
	sprintf(buf, "\r\n");
	send(client, buf, strlen(buf), 0);
	sprintf(buf, "\r\n");
	send(client, buf, strlen(buf), 0);
	sprintf(buf, "

	HTTP request method not supported.\r\n");
	send(client, buf, strlen(buf), 0);
	sprintf(buf, "\r\n");
	send(client, buf, strlen(buf), 0);
}

~~~

最后介绍一下startup函数，它启动server socket。

~~~java
int startup(u_short *port) {
	int httpd = 0;
	struct sockaddr_in name;

	//标准socket初始化过程
	httpd = socket(PF_INET, SOCK_STREAM, 0);
	if (httpd == -1)
		error_die("socket");
	memset(&name, 0, sizeof(name));
	name.sin_family = AF_INET;
	name.sin_port = htons(*port);
	name.sin_addr.s_addr = htonl(INADDR_ANY);
	if (bind(httpd, (struct sockaddr *) &name, sizeof(name)) < 0)
		error_die("bind");
	if (*port == 0) /* if dynamically allocating a port */
	{
		int namelen = sizeof(name);
		if (getsockname(httpd, (struct sockaddr *) &name, &namelen) == -1)
			error_die("getsockname");
		*port = ntohs(name.sin_port);
	}
	if (listen(httpd, 5) < 0)
		error_die("listen");
	return (httpd);
}
~~~

整个项目很简单，可以理解linux下的socket和常规的进程启动、通信的编写方法。