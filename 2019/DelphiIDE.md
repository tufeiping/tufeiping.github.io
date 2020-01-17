## 扩展Delphi IDE

> 原创 涂飞平  `2019-04-09`

<p style="text-align: center;"><img src="http://store.tufeiping.cn/tech.png" alt="tech.png"></p>


### 简介

Embarcadero Delphi® 提供丰富的API以便Delphi开发人员自定义和扩展自己的Delphi IDE。本文的目的就是通过介绍其中的部分API和示例，并且，还会有部分TMS公司提供的部分免费的IDE扩展的源码，给大家演示如何编写Delphi IDE扩展。

所有的扩展IDE的插件源码都是基于OTAPI（OpenTools API的简写）扩展接口的，该文所有的示例都是在Delphi XE版本下编写和测试的。

代码或者本文PDF版本可以联系译者获取。

该文档由涂飞平翻译，并放弃本中文版所有权利，这只是我这个假期的一个小任务，将其翻译后给技术人员参考。

原文参考： [ https://www.embarcadero.com/images/dm/technical-papers/extending-the-delphi-ide.pdf]( https://www.embarcadero.com/images/dm/technical-papers/extending-the-delphi-ide.pdf)

### 其他

除了本文，您还可以在Delphi XE安装目录的源码目录中找到更多关于OpenTools API内容的单元TOOLSAPI.PAS，具体位置为：

~~~text
Delphi安装路径\RAD Studio\8.0\source\ToolsAPI
~~~

TOOLSAPI.PAS单元不仅包含接口的定义,也有许多有用的注释, 可以帮助您了解各个接口的用途。在开发时可以参考这个有价值的内容。

另外，GExperts项目作为Delphi最成功的插件产品之一，其FAQ具有重要价值，可以找到很多插件开发常见问题的答案。

[http://www.gexperts.org/otafaq.html](http://www.gexperts.org/otafaq.html)

注意：插件和扩展本文可能会混用，不作特别区别。

### 基本架构

OpenTools API是基于接口实现的。接口命名通常以前缀IOTA或INTA（Native Tools API Interface简写）开头。IDE公开了很多可以从插件调用的接口；相反,当在IDE中触发特定操作时,IDE也可以调用您写的插件代码。为了在特定事件（操作）触发调用插件的代码，插件必须导出特定接口并注册给IDE，大多数情况下，这是通过编写从TNotifierObject继承的类完成的，该类需要实现接口并在IDE中注册该类。而作为插件编写者的您，都是编写调用IDE接口的代码以及编写被IDE调用的接口类的实现代码，来完成您的特定工作。

### IDE可扩展的区域

Delphi IDE本身也是VCL开发的，所以非常容易开放出接口。Delphi IDE可以通过多种方式进行扩展。也支持对几乎所有的区域进行定制，以下是扩展Delphi IDE最常见扩展区域：

● 创建停靠面板

添加自定义停靠面板,如组件面板、对象属性面板等。

● 与Code Editor交互

操作Delphi IDE的代码编辑器内容，如插件代码片段，替换文本，特定的快捷键，自定义语法高亮等操作。

● 与Code Insight交互

提供给代码编辑器自定义帮助上下文。

● 与Project Manager交互

提供给您定制工程管理面板的上下文菜单（右键菜单）的定制能力。

● 创建新类型的向导

添加Delphi新增自定义新类型或资源的向导。从这些向导中，可以创建新的项目类型，如特定的窗体类型或数据模块。

● 与TODO交互

与IDE的TODO列表项进行交互。

● 与调试器交互,创建自定义调试器可视化工具

在新版本的Delphi中，可以添加调试器扩展，以便在调试时提供特定数据类型的自定义显示。

● 与Form设计器交互

与Form设计器进行交互。

● 启动屏的提示设置

提供接口以便在Delphi IDE启动屏显示特定的信息，如插件信息。

### ToolsAPI的历史

随着Delphi不断的发展，扩展IDE的API也在不断发展。在Delphi XE中，继续扩展了原来用于自定义IDE的API，并添加了新的API以支持IDE的新功能。由于OTAPI主要是基于接口的，因此为了接口的延续性，通常都是通过新接口提供新功能。新接口是继承了老接口，这样就能同时兼容以前的接口，让插件能在当前版本及之前版本的IDE中运行。

Delphi团队采用了从早期接口不断增加版本号的接口命名方式来为新接口提供带有使用IDE版本后缀的接口名称。

注意，Delphi IDE版本与编译器版本是不同的。

例如,扩展项目向导的接口最初是IOTARepositoryWizard，在后来的Delphi版本中，它被扩展并命名为IOTARepositoryWizard60，更后来，又有了IOTARepositoryWizard80。

接口间都是通过继承来扩展功能，这点与类的继承是一样的，扩展的接口包含基础接口的所有方法，并加入了新的方法，这样可以有效的支持插件在之前版本的IDE中继续有效运行。

{最基础的向导接口定义}
~~~txt
IOTARepositoryWizard = interface(IOTAWizard)
~~~
 

{从Delphi 7开始提供的新的向导接口定义，相比原始接口，提供了一种区分VCL和CLX项目的方法}
~~~txt
IOTARepositoryWizard60 = interface(IOTARepositoryWizard)
~~~
 

{Delphi 8中引入了新的接口，除了提供Delphi7向导接口的功能，又提供了对IDE是VCL还是VCL.NET的判断}
~~~txt
IOTARepositoryWizard80 = interface(IOTARepositoryWizard60)
~~~
 

下面列出IDE的版本号，它们经常出现在OTA的接口名中，大家了解一下：
~~~txt
60 = Delphi 7

80 = Delphi 8

90 = Delphi 2005

100 = Delphi 2006

110 = Delphi 2007

120 = Delphi 2009

140 = Delphi 2010

145 = Delphi XE
~~~

### 连接IDE

编写Delphi IDE插件，大部分时候是插件操作IDE特定的部分和调用特定的功能。为了做到这一点，Delphi通过接口方式，公开了很多接口。当Delphi IDE启动时，如果您的代码中加入了ToolsAPI单元，Delphi会创建一个接口类型的全局变量BorlandIDEServices。这个变量提供给插件查询其他所需接口的能力。

例如

~~~pascal
// 检查BorlandIDEServices全局变量是否存在且可访问
if Assigned(BorlandIDEServices) then
begin

  {将接口全局变量转换为IOTAModuleServices（其实就是接口查询操作）并调用IOTAModuleServices的CloseAll方法关闭所有模块}
  (BorlandIDEServices as IOTAModuleServices).CloseAll;
end;
~~~
 

下面列出Delphi XE IDE常用接口

● INTAServices

可以访问IDE的工具栏，菜单，Imagelists，Actions


● IOTAActionServices

操作IDE的接口，如打开文件，关闭文件，保存文件等
 

● IOTACodeInsightServices

接口提供访问代码管理及其接口IOTACodeInsightManager
 

● IOTADebuggerServices

访问调试器相关的功能，如断电，事件日志，进程状态等
 

● IOTAEditorServices

访问IDE代码编辑器，代码缓存，选项等
 

● IOTAEditorViewServices

访问代码编辑器
 

● INTAEnvironmentOptionsServices

访问IDE的工具选项，菜单
 

● IOTAKeyBindingServices

访问快捷键绑定的接口
 

● IOTAKeyboardServices

访问IDE的宏录制及播放的接口
 

● IOTAMessageServices

访问IDE消息小窗口，可以加入自定义的消息窗口，删除消息等
 

● IOTAModuleServices

访问打开的模块文件的接口(如：工程，代码，Form文件等)
 

● IOTAPackageServices

访问已安装的包和组件列表的接口
 

● IOTAServices

访问Delphi自身的一些属性，如产品标识，安装目录，语言等信息
 

● IOTAToDoServices

访问某个模块TODO列表的接口
 

● IOTAWizardServices

获取向导的接口
 

● IOTAHighlightServices

语法高亮操作接口，可以添加自定义的语法高亮方案（通过接口IOTAHighlighter提供）
 

● IOTAPersonalityServices

访问IDE已关联的文件类型接口及关联文件扩展名的接口
 

● IOTACompileServices

提供操作编译器的接口，可以控制编译开始，结束以及编译完成后的反馈操作

### 开始创建IDE插件

Delphi IDE插件是安装到IDE中的包（Package）。Delphi将在启动过程中加载包。包既可以使用包单元中的Initialization代码段部分中执行的代码来执行其初始化，也可以提供在IDE中注册的Register方法。这与在IDE中安装组件包非常相似，在该包加载后，IDE从该包调用Register过程，RegisterComponents方法向IDE注册新组件。要安装IDE插件或扩展，至少需要创建一个类型为TNotifierObject的类,该类实现IOTAWizard接口并对其进行注册。
 
~~~pascal
unit MyIDEPlugin;

interface

uses
Classes, ToolsAPI;

TMyIDEWizard = class(TNotifierObject, IOTAWizard)
  //这里是需要实现的接口方法
end;

implementation

//注册到IDE
procedure Register;
begin
  RegisterPackageWizard(TMyIDEWizard.Create);
end;
~~~


没错，这就是第一个简单插件的全部，虽然它不提供任何实际功能，安装后，IDE将在需要时调用适当的接口方法。

### 扩展向导接口

<p style="text-align: center;"><img src="http://store.tufeiping.cn/wizard.png" alt="wizard.png"></p>

我们扩展IDE的第一个深入的例子是创建自定义向导。

若要创建向导插件，必须编写一个实现 `IOTAWizard` 接口和 `IOTARepositoryWizard` 接口的类。如上节内容所述，`IOTARepositoryWizard` 接口有多个版本。比如 Delphi 7 的 `IOTARepositoryWizard60` ，Delphi8的 `IOTARepositoryWizard80`，如果要操作新的版本，还是尽量实现新的接口。

在本例中,我们需要操作类别，所以要实现 `IOTARepositoryWizard80` 接口。

我们将编写一个从TNotifierObject继承的类，并且该类同时实现了 `IOTAWizard` 和 `IOTARepositoryWizard` 两个接口。

 ~~~pascal
TMyProjectWizard = class(TNotifierObject, IOTAWizard, IOTARepositoryWizard80)
  // IOTAWizard 接口方法
  function GetIDString: string;
  function GetName: string;
  function GetState: TWizardState;
  procedure Execute; 

  // IOTARepositoryWizard 接口方法
  function GetAuthor : string; 
  function GetComment : string;
  function GetPage : string;
  function GetGlyph: Cardinal; 

  // IOTARepositoryWizard80 接口方法
  function GetGalleryCategory: IOTAGalleryCategory;
  function GetPersonality: string;  
  function GetDesigner: string;
end;

// 注册插件
procedure Register;
begin
  RegisterPackageWizard(TMyProjectWizard.Create);
end;
~~~ 

当用户从IDE中的新建项目向导单击时，将调用 `Execute`方法。正是在此执行方法中，插件需要采取必要的步骤来创建所选项的源文件。在执行方法中，我们通过 `BorlandIDEServices` 全局变量获取 `IOTAModuleServices`接口，查询单元，类，文件等信息。然后使用`IOTAModuleServices`的`CreateModule`方法创建新模块并添加到当前项目中。

~~~pascal 
procedureTMyProjectWizard.Execute;
var
  LProj: IOTAProject;
  MS: IOTAModuleServices;
begin
  if Assigned(BorlandIDEServices) then
  begin
    MS := BorlandIDEServices as IOTAModuleServices;
    // 先查询IOTAModuleServices接口并调用其GetNewModuleAndClassName方法获取单元，类和文件信息
    MS.GetNewModuleAndClassName(‘’, FUnitIdent, FClassName, FFileName);
    FClassName := SFormName + Copy(FUnitIdent, 5, Length(FUnitIdent));
    LProj := GetActiveProject;
    if LProj <> nil then
    begin
      // 创建新的模块
      MS.CreateModule(TMyUnitCreator.Create(LProj, FUnitIdent, FClassName, FFileName));
    end;
  end;
end;
~~~
 

这里没有列出实现部分的代码，您可以在示例1中可以找到这个类的完整实现。
上述TMyUnitCreator类是一个实现`IOTACreator`,`IOTAModuleCreator`两个接口的类，通过这两个接口，可以创建实际的源码和`Form`文件。

~~~pascal
TMyUnitCreator = class(TNotifierObject, IOTACreator, IOTAModuleCreator)
public
  // IOTACreator
  function GetCreatorType: string;
  function GetExisting: Boolean; 
  function GetFileSystem: string;
  function GetOwner: IOTAModule; 
  function GetUnnamed: Boolean;

  // IOTAModuleCreator
  function GetAncestorName: string;
  function GetImplFileName: string;
  function GetIntfFileName: string; 
  function GetFormName: string; 
  function GetMainForm: Boolean;
  function GetShowForm: Boolean;
  function GetShowSource: Boolean;

  function NewFormFile(const FormIdent, AncestorIdent: string): IOTAFile;
  function NewImplSource(const ModuleIdent, FormIdent, AncestorIdent: string): IOTAFile;
  function NewIntfSource(const ModuleIdent, FormIdent, AncestorIdent: string): IOTAFile;
  procedure FormCreated(const FormEditor: IOTAFormEditor);
~~~
 

接口中最重要的方法是`NewFormFile`和`NewImplSource`方法。这两个函数应返回实现IOTAFile接口的类的实现，该接口将返回的`.DFM`窗体文件`.PAS`实际源代码文件。而`NewIntfSource`函数不适用于Delphi窗体或项目，仅用于C++头文件（适用于C++ Builder）。

`IOTAFile`是一个接口，它只是返回.DFM或.PAS文件的内容（作为一个字符串）和文件的修改时间（作为TDateTime）。

{TCodeFile类实现IOTAFile接口，这里由于除了IOTAFile之外，没有实现其他IOTA接口，所以继承自TInterfacedObject}
~~~pascal
TCodeFile = class(TInterfacedObject, IOTAFile)
protected
  function GetSource: string;
  function GetAge: TDateTime;
end;
~~~
 

请注意,这里返回的.DFM和.PAS文件内容必须是有效的代码。因为IDE会尝试分析窗体文件和源代码，如果分析失败，IDE将不会创建该文件。

在该例子中，单元和DFM内容都保存在资源文件里面（用BRCC32从.RC编译而来）。

TCompanyFormSRC 10 " Unit1.pas"

TCompanyFormFRM 10 " Unit1.DFM"

~~~pascal
function TCodeFile.GetSource: string;
var
  Text, ResName: AnsiString;
  ResInstance: THandle;
  HRes: HRSRC;
begin
  Resname := AnsiString(SCodeResName);
  ResInstance := FindResourceHInstance(HInstance);
  HRes := FindResourceA(ResInstance, PAnsiChar(ResName), PAnsiChar(10));
  Text := PAnsiChar(LockResource(LoadResource(ResInstance, HRes)));
  SetLength(Text, SizeOfResource(ResInstance, HRes));
  Result := Format(string(Text), [[FModuleName, FFormName, FAncestorName]]);
end;
~~~

### IDE停靠面板

为IDE创建自定义停靠面板是另一种类型的IDE扩展，我们不需要向IDE注册IOTAWizard接口实现类。

停靠面板插件需要创建窗体的实例，为用户提供一种显示或隐藏面板的方法，将其状态保留在IDE桌面上，在退出IDE时销毁停靠面板。

用于IDE停靠面板的类DockForm,DockToolForm是在作为DesignIDE包的一部分。存在两种不同的类型：TDockableForm和TDockableToolBarForm。我们需要在插件包的一个单元中编写一个Register过程。加载包后，IDE将调用此Register过程。通过此注册过程，我们可以创建停靠面板，并在IDE中插入一个新的菜单项，以显示/隐藏面板。当IDE关闭时，将调用单元的Finalization部分中的代码销毁面板。

创建自定义停靠面板的代码：

~~~pascal 
unit MyIDEDockPanel;

interface

procedure Register;  

implementation
uses
  MyDockForm;

var
  MyIDEDockForm: TMyDockForm;

procedure Register; 
begin
  if MyIDEDockForm = nil then 
  begin
    MyIDEDockForm := TMyIDEDockForm.Create(nil);
  end; 
end; 

finalization
  MyIDEDockForm.Free;
end.

~~~

TMyIDEDockForm是一个从TDockableForm或TDockableToolBarForm继承而来的类。要使用IDE桌面设置保存的功能，需要设置基类DeskSection,AutoSave和SaveStateNecessary几个属性，父类会与IDE桌面合作完成自动保存操作。

~~~pascal 
constructor TMyIDEDockForm.Create(AOwner: TComponent);
begin
  inherited;
  DeskSection := Name;
  AutoSave := True;
  SaveStateNecessary := True;
end; 

destructor TMyIDEDockForm.Destroy;
begin
  SaveStateNecessary := True;
  inherited; 
end;
~~~ 

此外,自定义停靠面板类应使用以下代码在IDE桌面上注册：
 
~~~pascal
RegisterDesktopFormClass(TMyIDEDockForm, MyIDEDockForm.Name, MyIDEDockForm.Name);

if @RegisterFieldAddress <> nil then
  RegisterFieldAddress(MyIDEDockForm.Name, @MyIDEDockForm);
~~~

运行后，显示如下：

<p style="text-align: center;"><img src="http://store.tufeiping.cn/dform.png" alt="dform.png"></p>

详细源码可以参考示例代码中DockForm文件夹内容。

### 连接代码编辑器

本节我们编写一个操作代码编辑器的插件。通过在BorlandIDEServices全局变量查询IOTAEditorServices接口，可以访问IDE代码编辑器。通过接口方法IOTAEditorServices.TopView: IOTAEditView可以对当前活动的编辑器视图进行访问。

而返回的IOTAEditView接口提供对编辑器书签、光标位置、滚动等要素的访问。另外，IOTAEditView还提供IOTAEditView.Buffer对编辑器代码缓冲区的访问。缓冲区可以通过接口IOTAEditBuffer操作文本，例如插入和删除。

 

下面代码片段还是从BorlandIDEServices中获取IOTAEditorServices，然后获取IOTAEditView接口并访问IOTAEditBuffer，在活动编辑器窗口中的源代码文件顶部插入一些文本：

 
~~~pascal
var
  EditorServices: IOTAEditorServices;
  EditView: IOTAEditView;
  copyright: string;
begin
  copyright := ‘Copyright © 2011 by tmssoftware.com';
  EditorServices := BorlandIDEServices as IOTAEditorServices;
  EditView := EditorServices.TopView;
  if Assigned(EditView) then
  begin
    // 插入点设置到页首
    EditView.Buffer.EditPosition.Move(1,1);
    // 插入文本
    EditView.Buffer.EditPosition.InsertText(copyright);
end;
~~~

### 扩展IDE菜单

Delphi IDE提供了IOTAMenuWizard接口用于添加Delphi IDE主菜单中的菜单项。当然，此接口有个限制：以这个接口方式添加的所有菜单项都将在“帮助”菜单下进行组织和显示。当然，我们也将提供一种替代方法，可以将我们的菜单显示在现有的Delphi IDE主菜单中的任何位置。

若要通过IOTAMenuWizard添加新的菜单项,，请创建一个TNotifierObject的子类，并实现IOTAWizard，IOTAMenuWizard两个接口。

~~~pascal
TMyMenuItem = class(TNotifierObject, IOTAWizard, IOTAMenuWizard)
  function GetIDString: string;
  function GetName: string;
  function GetState: TWizardState;
  procedure Execute;
  function GetMenuText: string;
end;
~~~
 

当然，还是需要Register过程，将其注册到IDE里面：

~~~pascal 
procedure Register;
begin
  RegisterPackageWizard(TMyMenuItem.Create);
end;
~~~
 

在IDE中单击菜单项时，Delphi将调用您接口中的Execute方法来完成事件，注意，这种方式添加的菜单在卸载插件时候不需要特殊处理，IDE会自动删除注册的菜单项。

下面介绍另外一种方法，这种方法可以将菜单放置在Delphi IDE菜单的任何位置。这种方法将直接访问Delphi IDE主菜单（VCL中的TMainMenu类），没错，Delphi IDE也是由VCL编写的，我们就可以使用TMainMenu方法在此菜单中插入TMenuItem实例。

从BorlandIDEServices中查询INTAServices40接口（这是一个比较早的接口，提供的也是比较底层的操作），并调用其返回TMainMenu实例的函数MainMenu。

~~~pascal
procedure AddIDEMenu;
var
  NTAServices: INTAServices40;
begin
  NTAServices := BorlandIDEServices as INTAServices40;
  if NTAServices.MainMenu.Items[5].Find('INTAServices40Menu') = nil then 
  begin
    CustomMenuHandler := TCustomMenuHandler.Create; 
    mnuitem :=  TMenuItem.Create(nil); 
    mnuitem.Caption := 'INTAServices40Menu'; 
    mnuitem.OnClick := CustomMenuHandler.HandleClick;
    NTAServices.MainMenu.Items[5].Add(mnuitem)
  end; 
end;
~~~
 

注：这种方式添加的菜单，在插件卸载的时候，需要自己删除菜单项。

~~~pascal
procedure RemoveIDEMenu;
var
  NTAServices: INTAServices40;
begin
  if Assigned(mnuitem) then 
  begin
    NTAServices := BorlandIDEServices as INTAServices40; 
    NTAServices.MainMenu.Items[5].Remove(mnuitem);
    mnuitem.Free;
    if Assigned(CustomMenuHandler) then
      CustomMenuHandler.Free;
  end; 
end;
~~~
 
详细源码可以参考示例代码中IDEMenu文件夹内容。

### 扩展工程管理窗口右键菜单

在本节中，我们将访问IDE工程管理及其上下文菜单，在工程管理中打开的项目和文件。此外，我们将使用自定义操作来扩展上下文菜单。

首先,我们将使用BorlandIDEServices全局变量中提供的IOTAProjectManager接口及接口的AddMenuItemCreatorNotifier方法，该方法需要返回一个继承自TNotifierObject的类，并实现IOTAProjectMenuItemCreatorNotifier接口的对象。IDE会在工程管理器上下文菜单中显示菜单并在点击后调用我们的插件代码，代码如下。

~~~pascal
TMyProjectContextMenu = class(TNotifierObject, IOTAProjectMenuItemCreatorNotifier)
  procedure AddMenu(const Project: IOTAProject; const IdentList: TStrings; const ProjectManagerMenuList: IInterfaceList; IsMultiSelect: Boolean);
end;

procedure TMyProjectContextMenu.AddMenu(const Project: IOTAProject; const IdentList: TStrings; const ProjectManagerMenuList: IInterfaceList; IsMultiSelect: Boolean); 
var
  MnuItem: TMyProjectContextMenuLocal;
begin
  MnuItem := TMyProjectContextMenuLocal.Create; 
  ProjectManagerMenuList.Add(MnuItem);
end;
~~~ 

其中，参数IdentList是一个字符串列表，其中包含右键单击的项目类型的字符串标识符。这在TOOLSAPI.PAS有下列定义： 

~~~pascal
sBaseContainer = ‘BaseContainer'; 

sFileContainer = ‘FileContainer'; 

sProjectContainer = ‘ProjectContainer';

sProjectGroupContainer = ‘ProjectGroupContainer'; 

sCategoryContainer = ‘CategoryContainer'; 

sDirectoryContainer = ‘DirectoryContainer'; 

sReferencesContainer = ‘References'; 

sContainsContainer = ‘Contains'; 

sRequiresContainer = ‘Requires';
~~~ 

如果您的上下文菜单项只出现在特定节点（比如工程组节点，工程节点，文件节点等）的右键菜单中，可以通过IdentList来判断：

~~~pascal
procedure TMyProjectContextMenu.AddMenu(const Project: IOTAProject; const IdentList: TStrings; const ProjectManagerMenuList: IInterfaceList; IsMultiSelect: Boolean); 
var
  MnuItem: TMyProjectContextMenuLocal;
begin
  // 只有在项目组右键菜单点击时候才添加我们的菜单项
  if (IdentList.IndexOf(sProjectGroupContainer) <> -1) then
  begin
    MnuItem := TMyProjectContextMenuLocal.Create;
    // Set menu item properties here 
    MnuItem.OnExecute := MenuClickHandler; 
    ProjectManagerMenuList.Add(MnuItem)
  end; 
end;
~~~
 

这里TMyProjectContextMenuLocal类是一个继承自TProjectContextMenuLocal类并实现IOTALocalMenu和IOTAProjectManagerMenu接口

~~~pascal 
TMyProjectContextMenuLocal = class(TProjectContextMenuLocal, IOTALocalMenu, IOTAProjectManagerMenu)
public
  // IOTALocalMenu
  function GetCaption: string;
  function GetChecked: Boolean;
  function GetEnabled: Boolean;
  // IOTAProjectManagerMenu interface
  function GetIsMultiSelectable: Boolean;
  procedure SetIsMultiSelectable(Value: Boolean);
  procedure Execute(const MenuContextList: IInterfaceList); overload;
  function PreExecute(const MenuContextList: IInterfaceList): Boolean;
  function PostExecute(const MenuContextList: IInterfaceList): Boolean;
  property IsMultiSelectable: Boolean read GetIsMultiSelectable write SetIsMultiSelectable;
end;
~~~ 

使用IOTALocalMenu接口可以设置菜单项的标题、选中状态和启用状态。使用IOTAProjectManagerMenu接口，我们可以定义上下文菜单项是否支持在工程管理器中的多个选定项上执行。还有一些支持方法，比如执行前，执行后处理，是否支持多选等。

~~~pascal
procedure TMyProjectContextMenuLocal.Execute(const MenuContextList: IInterfaceList);
var
  MenuContext: IOTAProjectMenuContext; 
  Project: IOTAProject;
begin
  MenuContext := MenuContextList.Items[0] as IOTAProjectMenuContext;
  Project := MenuContext.Project;
  // 对Project的其他操作
end;
~~~

<p style="text-align: center;"><img src="http://store.tufeiping.cn/dmenu.png" alt="dmenu.png"></p>

详细源码可以参考示例代码中ProjMenu文件夹内容。

### IDE代码编辑器的上下文菜单及选择文本上下文菜单扩展

如果您的插件希望在编辑器中对选定的文本提供一些处理，比如希望在IDE编辑器上下文菜单中添加一个自定义菜单项，单击该菜单项时，会抓取选定的文本并对其进行处理。

要添加这样的上下文菜单，我们需要获得对菜单的访问权限，而菜单又首先需要访问编辑器。请注意，在IDE启动时，编辑器无法立即使用，因此我们无法在插件初始化代码中访问编辑器实例。我们需要注册一个实现状态通知 INTAEditServicesNotifier接口的类。当编辑器在IDE中激活时，IDE将调用此接口告知我们，编辑器处于可用状态。INTAEditServicesNotifier接口提供了几种方法，其中EditorViewActivated是我们需要的。
 
~~~pascal
TEditNotifierHelper = class(TNotifierObject, IOTANotifier, INTAEditServicesNotifier)
  procedure WindowShow(const EditWindow: INTAEditWindow; Show, LoadedFromDesktop: Boolean);
  procedure WindowNotification(const EditWindow: INTAEditWindow; Operation: TOperation);
  procedure WindowActivated(const EditWindow: INTAEditWindow);
  procedure WindowCommand(const EditWindow: INTAEditWindow; Command, Param: Integer; var Handled: Boolean);
  procedure EditorViewActivated(const EditWindow: INTAEditWindow; const EditView: IOTAEditView);
  procedure EditorViewModified(const EditWindow: INTAEditWindow; const EditView: IOTAEditView);
  procedure DockFormVisibleChanged(const EditWindow: INTAEditWindow; DockForm: TDockableForm);
  procedure DockFormUpdated(const EditWindow: INTAEditWindow; DockForm: TDockableForm);
  procedure DockFormRefresh(const EditWindow: INTAEditWindow; DockForm: TDockableForm);
end;
~~~
 

向IDE注册编辑器窗口通知响应类的过程如下：

~~~pascal
procedure Register; 
var
  Services: IOTAEditorServices;
begin
  Services := BorlandIDEServices as IOTAEditorServices; 
  NotifierIndex := Services.AddNotifier(TEditNotifierHelper.Create);
end;
~~~
 

函数中NotifierIndex是AddNotifier的返回值。在卸载插件时，需要使用将NotifierIndex卸载。

~~~pascal
procedure RemoveNotifier;
var
  Services: IOTAEditorServices;
begin
  if NotifierIndex <> -1 then 
  begin
    Services := BorlandIDEServices as IOTAEditorServices; 
    Services.RemoveNotifier(NotifierIndex);
    NotifierIndex := -1;
  end; 
end;
 
finalization
  RemoveNotifier; // 终止后，自动注销注册的Notifier
end.
~~~
 

安装此通知插件后，现在可以在编辑器首次处于活动状态并安装自定义菜单项时访问IDE编辑器上下文菜单，这个时候访问编辑器就是安全的了。

~~~pascal
var
custommenu: TMenuItem;

procedure TEditNotifierHelper.EditorViewActivated(const EditWindow: INTAEditWindow; const EditView: IOTAEditView); 
begin
  if not Assigned(custommenu) then 
  begin
    AddEditContextMenu;
  end; 
end;

procedure AddEditContextMenu;
var
  editview: IOTAEditView140; 
  popupmenu: TPopupMenu;
begin
  editview := (BorlandIDEServices as IOTAEditorServices).TopView; 
  popupmenu := editview.GetEditWindow.Form.FindComponent(‘EditorLocalMenu') as TPopupMenu;
  custommenu := TMenuItem.Create(nil); 
  custommenu.Caption := ‘Custom context menu item'; 
  custommenu.OnClick := mnuHandler.MenuHandler; 
  popupmenu.Items.Add(custommenu);
end;

initialization
  custommenu := nil; 

finalization
  custommenu.Free;
end.
~~~
 

代码中的mnuHandler是TMenuHandler的实例，TMenuHandler类及实现代码如下
 
~~~pascal
TMenuHandler = class(TComponent) 
  procedure MenuHandler(Sender: TObject);
end;

procedure TMenuHandler.MenuHandler(Sender: TObject);
var
  editview: IOTAEditView140; 
  editblock: IOTAEditBlock;
begin
  editview := (BorlandIDEServices as IOTAEditorServices).TopView;
  // get the selected text in the edit view
  editblock := editview.GetBlock;
  ShowMessage(‘Context menu click: ' + inttostr(editblock.StartingColumn)+‘:'+inttostr(editblock.Starting Row) + ‘ - ' + inttostr(editblock.EndingColumn)+ ‘:'+inttostr(editblock.EndingRow));
  // if there is a selection of text, get it via editblock.Text
  if (editblock.StartingColumn <> editblock.EndingColumn) or (editblock.StartingRow <> editblock.EndingRow) then
    ShowMessage(‘Selected text: ' + editblock.Text);
end;
~~~
 

详细源码可以参考示例代码中EditorContextMenu文件夹内容。

### IDE启动屏添加信息

有时候，我们需要给Delphi IDE启动屏设置一些提示信息，比如告诉用户已经安装了我们的插件。TOOLSAPI单元暴露了SplashScreenServices: IOTASplashScreenServices变量给我们使用。这个接口包含AddPluginBitmap方法，是不是感觉这个名字很怪，难道只是加位图吗？其实这个方法就可以完成全部操作了，我们这节就只需要这个方法。
 
~~~pascal
procedure AddPluginBitmap(const ACaption: string; ABitmap: HBITMAP; AIsUnRegistered: Boolean = False; const ALicenseStatus: string = ''; const ASKUName: string = '');
~~~ 

参数说明：

~~~text
ACaption 要显示在初始屏幕上的提示文本

ABitmap LOGO位图资源句柄（请使用24x24像素的尺寸）

AIsUnRegistered 插件是否注册，它会影响字体颜色，未注册产品的文本以红色字体显示,已注册的产品显示常规白色文本

ALicenseStatus 授权信息，一般就是'Trial'或'Registered'

ASKUName 如果插件存在不同的SKU,则此文本可以显示SKU的名称
~~~
 

示例
~~~pascal
procedure AddSplashText;
var
  bmp: TBitmap;
begin
  bmp := TBitmap.Create;
  bmp.LoadFromResourceName(HInstance, ‘PLUGINBITMAPRESOURCE');
  SplashScreenServices.AddPluginBitmap(‘Plugin product XYZ © 2011 by MyCompany', bmp.Handle, false, ‘Registered', '');
  bmp.Free;
end;

initialization
  AddSplashText;
end.
~~~
 

详细源码可以参考示例代码中SplashScreen文件夹内容。

### TMS开放的几个免费的DELPHI XE插件

基于本文介绍的技术，我们为Delphi XE提供了几个免费的IDE插件。免费插件可以在代码下载的文件夹“Delphi XE的TMS插件”中找到。

### TMS工程管理器插件

该插件提供工程管理器3个上下文菜单选项：


1)ZIP project: 将工程文件压缩为ZIP包

2)ZIP & Email project: 压缩工程并发送Email

3)ZIP & Upload project: 压缩工程代码并上传到指定FTP服务器

<p style="text-align: center;"><img src="http://store.tufeiping.cn/ddemo.png" alt="ddemo.png"></p>

（截图居然只有两个选项 :-(）

### TMS新插件浏览

这是一个停靠窗口类型的插件。插件包含3个选项页签，显示TMS最新发布的组件，博客订阅信息和推特信息。

<p style="text-align: center;"><img src="http://store.tufeiping.cn/dtms.png" alt="dtms.png"></p>

### TMS演示工具

TMS演示工具插件是基于停靠窗口插件功能以及IDE编辑器的接口。它提供了两个选项页签。

第一个选项卡中是剪贴板监视器。这将显示放在剪贴板上的所有文本。您可以直接将停靠窗体内容拖放到IDE编辑器。

第二个选项卡是已保存代码段的列表。在演示文稿时，可以使用这些代码片段。

<p style="text-align: center;"><img src="http://store.tufeiping.cn/dplugin.png" alt="dplugin.png"></p>

### 关于作者

#### 布鲁诺（Bruno）

在根特大学学习电子工程，并在比利时Barco Graphics开始了数字硬件研发工程师的职业生涯。

他于1996年创建了TMS软件，从Delphi 1开始开发VCL组件。

TMS软件于1998年成为Borland技术合作伙伴，并开发了Delphi Informant屡获殊荣的Grid和Scheduling组件。

2001年，他开始开发Intraweb组件。

2003年，他开始开发ASP.NET组件。

目前，布鲁诺正在开发和管理VCL、Silverlight、ASP.NET和IntraWeb组件开发项目，使用Delphi和XCode在Windows、Web和IPad上进行咨询和自定义项目开发和管理。

他特别感兴趣的领域是用户接口和硬件。



#### Embarcadero Technologies,Inc.是软件工具的领先提供商，它使应用程序开发人员和数据管理专业人员能够在异构IT环境中更高效地设计、构建和运行应用程序和数据库。在财富100强公司中，超过90%的公司和全球300多万用户的活跃社区依靠Embarcadero屡获殊荣的产品来优化成本、简化合规性并加快开发和创新。

Embarcadero成立于1993年，总部位于旧金山,在世界各地设有办事处。

[http://www.embarcadero.com](http://www.embarcadero.com)

Copyright©2011 TMS software

<p style="text-align: center;"><img src="http://store.tufeiping.cn/ddyonyou.png" alt="dyonyou.png"></p>

> 涂飞平 2019-04-06 于北京用友产业园