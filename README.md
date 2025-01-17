
在一个业务管理系统中，如果我们需要实现权限控制功能，我们需要定义好对应的权限功能点，然后在前端界面中对界面元素的可用性和功能点进行绑定，这样就可以在后台动态分配权限进行动态控制了，一般来说，权限功能点是针对角色进行控制的，也就是简称RBAC（Role Based Access Control）。对于登录系统后的用户，对用户的菜单（工具栏）、界面操作按钮的权限进行动态化的绑定和统一处理的操作过程，这样对于我们界面，只需要约定一些规则即可实现比较弹性化的操作，非常方便。本篇随笔介绍WxPython跨平台开发框架之动态菜单的管理和功能权限的控制。


### 1、权限管理系统的相关资源


权限管理系统主要的功能包括有：用户管理、组织机构管理、功能管理、角色管理和权限分配管理、菜单管理、系统类型管理、登录日志管理、操作日志管理、系统黑白名单管理等功能模块。对于每新增一个系统，我们只需要在权限管理系统中增加一个系统类型定义，以及相关的功能、菜单数据即可，非常方便管理。下图是一个简化的权限管理系统中涉及到角色相关资源的信息。


![](https://img2024.cnblogs.com/blog/8867/202412/8867-20241231132329901-381700152.png)


菜单资源和权限功能点是基于不同系统前端定义的资源，因此可以一套系统管理多个终端的菜单、功能点，以便实现更好的控制。


权限系统分了两级管理员用户：超级管理员和公司管理员。超级管理员可以管理整个集团或者整个系统的人员和相关信息（包括组织机构、角色、登陆日志、操作日志等信息的分级）；公司管理员可以管理分子公司、事业单位处室/局级这样的组织机构的人员和相关信息。分级管理组织机构、角色、用户等相关数据，能够减少管理员的相关工作，提高工作效率，并能增强权限管理系统对权限的控制和资源分配等管理，提高用户的认同感。


#### 1）功能点定义和管理


权限功能点的管理就是对TB\_Function的表的管理操作，这个表是我们定义用于系统控制的功能点。


权限功能点的管理为了展示它的树状结果，包括树列表的管理和明细列表的管理，如下图所示。


![](https://img2024.cnblogs.com/blog/8867/202412/8867-20241231133029024-1544495257.png)


我们为了方便，在开始的时候，创建功能点的时候，一般通过批量添加的方式快速添加，如下界面所示。


![](https://img2024.cnblogs.com/blog/8867/202412/8867-20241231133155400-793975398.png)


这样系统会根据主控制标识，为各个操作（增加、删除、修改等）增加对应的操作标识。


![](https://img2024.cnblogs.com/blog/8867/202412/8867-20241231133305040-623690116.png)


 为对应角色添加相关的操作功能点


![](https://img2024.cnblogs.com/blog/8867/202412/8867-20241231133431168-1570178871.png)


 


#### 2）菜单资源的定义和管理


一般的业务系统，需要对菜单进行动态配置管理，通过后台菜单的配置和权限的指定，能够实现菜单的动态加载和权限验证。因此菜单也是权限分配的一部分，为了有效管理菜单资源，我们把菜单放到权限管理系统中进行管理控制，可根据用户权限进行动态控制显示。


菜单的管理界面如下所示。


![](https://img2024.cnblogs.com/blog/8867/202501/8867-20250102095805877-1537001107.png)


 菜单信息，包括名称和相关的图标定义等，这些需要再前端界面中构建工具栏显示用到的资源，而窗体类型则是定义我们需要动态展示的菜单对象的模块名和类名称。


![](https://img2024.cnblogs.com/blog/8867/202501/8867-20250102100001791-255720996.png)


 菜单定义好后，为了和实际用户进行关联，那么需要为角色添加相关的访问菜单，从而实现用户菜单的动态关联。


![](https://img2024.cnblogs.com/blog/8867/202412/8867-20241231133548224-1489136460.png)


我们在开发初期，模拟定义了静态的菜单资源的信息，如下所示。




```
class ToolbarUtil:
    """工具栏菜单的创建类"""

    @staticmethod
    def create_tools():
        """创建工具栏菜单嵌套集合"""
        menus = [
            MenuInfo(
                id="01",
                label="用户管理",
                icon="user",
                path="views.frm_user.FrmUser",
            ),
            MenuInfo(
                id="02",
                label="组织机构管理",
                icon="organ",
                path="views.frm_ou.FrmOU",
            ),
            ...............
        ]
        return menus
```


那么我们有了动态定义的菜单和动态分配的功能后，我们就可以根据用户ID（角色ID）从后端系统接口中获得对应的菜单列表，然后统一展示即可。




```
    @staticmethod
    def create_tools_dynamic():
        """动态创建工具栏菜单"""

        # 同步获取菜单信息
        menus = api_menu.get_all_nodes_by_user_sync(
            settings.CurrentUser.id, settings.SystemType
        )
        # 定义字段映射
        field_mapping = {"name": "label", "winformtype": "path", "icon": "icon"}
        newmenus = map_with_dynamic_alias(menus, MenuInfo, field_mapping)

        return newmenus
```


这样我们在主窗体界面中，构建菜单的函数如下所示。




```
    def _create_toolbar(self, d="H"):
        """创建工具栏"""
        agwStyle = aui.AUI_TB_TEXT | aui.AUI_TB_OVERFLOW

        if d.upper() in ["V", "VERTICAL"]:
            agwStyle = aui.AUI_TB_TEXT | aui.AUI_TB_VERTICAL

        tb = aui.AuiToolBar(
            self, -1, wx.DefaultPosition, wx.DefaultSize, agwStyle=agwStyle
        )
        tb.SetToolBitmapSize(wx.Size(16, 16))

        # 动态创建工具栏
        toolbars = ToolbarUtil.create_tools_dynamic()

        for item in toolbars:
            tool_id = wx.NewIdRef()
            bitmap = get_bitmap(item.icon)
            tb.AddSimpleTool(
                tool_id=tool_id,
                label=item.label,
                bitmap=bitmap,
                short_help_string=item.tips if item.tips else item.label,
            )

            # 绑定事件
            self.Bind(
                wx.EVT_TOOL,
                partial(self.on_tool_event, item),  # 这里传递菜单信息
                id=tool_id,
            )

        # 增加系统常用按钮
        tb.AddSeparator()
        tb.AddSimpleTool(
            self.id_close_all, "关闭所有", images.delete_all.Bitmap, "关闭所有页面"
        )
        tb.AddSimpleTool(self.id_quit, "退出", images.close.Bitmap, "退出程序")

        self.Bind(
            wx.EVT_TOOL, lambda event: EventPub.show_about_dialog(), id=self.id_about
        )
        self.Bind(
            wx.EVT_TOOL, lambda event: EventPub.close_all_page(), id=self.id_close_all
        )
        tb.Realize()
        return tb
```


动态构建部分菜单后，并加上一些额外的操作功能项目即可。


要动态构建视图的实例，我们需要用到importlib库来导入模块，然后在模块中获得对应类型进行构造处理即可。




```
import importlib
import wx

"""要自动导入一个给定的类（比如 "views.testaui_panel.DocumentPanel"）并在 Python 中使用它，
可以利用 importlib 库来动态地导入模块和类。importlib 允许你在运行时加载模块，
而不需要在代码中显式地使用 import 语句。"""

def dynamic_import(class_path: str):
    # 拆分字符串为模块路径和类名
    # print(f"Dynamic import: {class_path}")
    module_path, class_name = class_path.rsplit(".", 1)

    # 动态导入模块
    module = importlib.import_module(module_path)

    # 获取类对象
    class_obj = getattr(module, class_name)

    # 返回类对象
    return class_obj
```


例如可以通过下面类似的代码实现对话框展示或者窗口的展示。




```
def show_window(class_path: str, parent=None, **kwargs):
    """根据路径和基类对象，显示窗口"""
    window_class = dynamic_import(class_path)
    window = window_class(parent, **kwargs)
    # 判断传入的窗口是 wx.Dialog 还是 wx.Panel
    if isinstance(window, wx.Dialog):
        # 如果是 wx.Dialog，调用 ShowModal
        result = window.ShowModal()
        print(f"Dialog closed with result: {result}")
        window.Destroy()  # 在对话框关闭后销毁
    elif isinstance(window, wx.Panel):
        # 如果是 wx.Panel，调用 Show
        window.Show()
        print("Panel shown")
    else:
        print("Unknown window type")
```


 


### 2、功能点和按钮的控制处理


在上面的菜单资源中，直接和角色关联，在用户登录系统后，自动构建对应的菜单（工具栏）显示，而对于功能点和按钮的关联控制处理，我们也是可以采用类似的方式处理的。


首先我们在当前终端的全局设置对象里面定义好拥有的功能点对象，如下代码所示。


![](https://img2024.cnblogs.com/blog/8867/202501/8867-20250102101803178-978554576.png)


我们在用户成功登录系统（认证用户通过）后，获取用户的详细信息，以及相关联的资源（包括菜单、功能点、角色列表）等信息进行全局存储，方便在用到的地方进行调用判断，如下代码所示。




```
    async def SetLoginInfo(self):
        """设置登录信息"""
        userid = settings.AccessTokenResult.userid
        res = await api_user.Get(userid)
        if res.success:
            settings.CurrentUser = res.result
            await self.GetSystemType()
            await self.GetFunctionsByUser(userid)
            await self.GetRolesByUser(userid)

            EventPub.user_info_loaded()  # 发布用户信息加载完成事件
```


我们前面随笔介绍过，一般列表窗体是继承一个统一的基类的。


在我的WxPython跨平台开发框架中，我们对于常规窗体列表界面做了抽象处理，一般绝大多数的逻辑封装在基类上，基类提供一些可重写的函数给子类实现弹性化的处理。


如下是基类窗体和其他窗体之间的集成关系。


![](https://img2024.cnblogs.com/blog/8867/202411/8867-20241111131058275-415274506.png)


由于基类是通过泛型类型的定义的，因此可以对子类的相关逻辑进行统一抽象处理，以便实现常规功能的控制（包括新增、编辑、删除、批量添加、打印、导入、导出等）。


如对于客户信息的列表窗体，我们的视图类如下所示（（通过代码生成工具Database2Sharp生成即可，之前随笔《[在自家的代码生成工具中，增加对跨平台WxPython项目的前端代码生成，简直方便的不得了](https://github.com/wuhuacong/p/18583832)》介绍过）。


![](https://img2024.cnblogs.com/blog/8867/202501/8867-20250102102535184-1162207911.png)


 我们可以看到，其中model是由子类传入的一个对象类型，那么我们在父类也可以统一进行获取它的名称进行处理即可。


在BaseListFrame类里面，我们定义一个判断是否有对应功能点的函数，如下所示。




```
    def HasAction(self, action_id: str, entity_name: str = None) -> bool:
        """判断权限缓存是否存在指定操作功能id

        :param action_id: 操作功能id, 如Add,Edit,Delete, Import,Export等
        :param entity_name: 实体名称, 默认根据self.model的名称获取并替换后缀Dto，如Customer,Product等
        :return: 存在返回True，否则返回False
        """
        if not action_id:
            return False

        if not entity_name:
            # 根据self.model的名称获取并替换后缀Dto
            entity_name = self.model.__qualname__.replace("Dto", "")

        # 判断权限缓存是否存在类似"Customer:Add"格式的字符串
        result = f"{entity_name}:{action_id}" in settings.FunctionDict
        return result
```


这样我们约定了模块和功能点的名称前缀后，就可以通用的处理判断了。


![](https://img2024.cnblogs.com/blog/8867/202501/8867-20250102103549250-1271720731.png)


由于我们常规的新增、编辑、删除、导出等操作由父类统一生成标准的按钮，那么我们就可以根据是否有某些功能的标识进行构建了，如下代码所示。




```
    def _CreateCommonButtons(self, pane: wx.Window):
        """父类窗口统一创建通用按钮"""
        # 增加按钮
        btns_sizer = wx.BoxSizer(wx.HORIZONTAL)

        button_list = []
        # 设置图标和位置
        if EventFlags.EVT_Search & self.EVT_FLAGS:
            btn_search = ControlUtil.create_button(
                pane,
                "查询",
                "search",
                handler=self._on_first_page,
                is_async=True,
                id=wx.ID_FIND,
            )
            button_list.append(btn_search)
        if self.has_add:
            btn_add = ControlUtil.create_button(
                pane, "新增", "add", handler=self.OnAdd, is_async=True
            )
            button_list.append(btn_add)
        if self.has_edit:
            btn_edit = ControlUtil.create_button(
                pane, "编辑", "edit", handler=self.OnEdit, is_async=True
            )
            button_list.append(btn_edit)
        if self.has_delete:
            btn_delete = ControlUtil.create_button(
                pane, "批量删除", "delete", handler=self.OnDelete, is_async=True
            )
            button_list.append(btn_delete)
        if self.has_export:
            btn_export = ControlUtil.create_button(
                pane, "导出Excel", "xls", handler=self.OnExport, is_async=True
            )
            button_list.append(btn_export)

        # 将按钮添加到按钮组中
        for btn in button_list:
            btns_sizer.Add(btn, 0, wx.ALL, 3)

        return btns_sizer
```


我们这里使用了辅助类创建按钮 ControlUtil.create\_button 方便控制相关的内容。类似的右键菜单我们也可以如此操作，判断权限是否拥有再构建即可。




```
    def _on_showmenu(self, event: wx.grid.GridEvent) -> None:
        """父类窗体的右键菜单处理"""

        # 创建右键菜单对象
        menu: wx.Menu = wx.Menu()

        if self.has_add:
            ControlUtil.create_menu(
                self, menu, "新增", icon_name="add", handler=self._OnMenuAdd
            )
        if self.has_edit:
            ControlUtil.create_menu(
                self, menu, "编辑", icon_name="edit", handler=self._OnMenuEdit
            )
        if self.has_delete:
            ControlUtil.create_menu(
                self, menu, "删除选中行", icon_name="delete", handler=self._OnMenuDelete
            )

        if self.has_export:
            ControlUtil.create_menu(
                self, menu, "导出Excel", icon_name="xls", handler=self._OnMenuExport
            )

          ............
```


![](https://img2024.cnblogs.com/blog/8867/202501/8867-20250102105712753-1546984116.png)


这些都是父类窗体，对通用操作的权限判断和创建处理，如果对于子类窗体，我们也可以使用这些判断标识来增加一些额外的操作按钮或者菜单的。


如对于字典模块列表界面中，判读它是否有批量添加的操作权限，并添加相关的功能入口。如下所示。




```
    def CreateCustomMenus(self, parent_menu: wx.Menu) -> None:
        """子类重写该方法，创建自定义菜单"""
        # 父类已创建默认菜单，这里添加自定义菜单
        if self.has_batch_add:
            ControlUtil.create_menu(
                self,
                parent_menu,
                "批量添加字典",
                "batch_add",
                handler=self.OnMenuBatchAdd,
            )
```


对于查看/编辑/新增的窗体，它们也是有一个通用的编辑对话框基类的，因此也可以和列表的方式实现同样的功能控制，这里不在赘述。


以上就是对于登录系统后的用户，对用户的菜单（工具栏）、界面操作按钮的权限进行动态化的绑定和统一处理的操作过程，这样对于我们界面，只需要约定一些规则即可实现比较弹性化的操作，非常方便。


 


 本博客参考[milou加速器](https://xinminxuehui.org)。转载请注明出处！
