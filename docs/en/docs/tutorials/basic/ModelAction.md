# Model Action

 Model management actions refer to operations performed on one or more model data such as the most basic operations CRUD. But often you may need to add some special operations. For example: change data state or perform certain tasks. In this case you can add custom model management actions.
`fastapi_amis_admin` has various types of model actions, the following briefly demonstrates several possible common action methods.

## Custom Toolbar Actions

### example-1

```python
@site.register_admin
class ArticleAdmin(admin.ModelAdmin):
    page_schema = PageSchema(label='page_schema', icon='fa fa-file')
    model = Article

    # Adding custom toolbar actions
    admin_action_maker = [
        lambda admin: AdminAction(
            admin=admin,
            name='test_ajax_action',
            action=ActionType.Ajax(
                label='toolbar_ajax_action',
                api='https://3xsw4ap8wah59.cfc-execute.bj.baidubce.com/api/amis-mock/mock2/form/saveForm'
            ),
            flags=['toolbar']
        ),
        lambda admin: AdminAction(
            admin=admin,
            name='test_link_action',
            action=ActionType.Link(
                label='toolbar_link_action',
                link='https://github.com/amisadmin/fastapi_amis_admin'
            ),
            flags=['toolbar']
        )
    ]

```

In this example, through `admin_action_maker` field - two model actions are added to the toolbar:

1. `ActionType.Ajax` this action will send ajax request to the specified API.
2. `ActionType.Link` after clicking this action it will jump to the specified link.

!!! note annotate "About `ActionType`"

    ActionType - is actually a python model mapping of [amis Action action button](https://baidu.gitee.io/amis/zh-CN/components/action?page=1) component, which supports varienty of common action types. For example: ajax request; dowload request; jump link; send email; pop-up window; drawer; copy text; etc.
    
   Strong flexibility of fastapi_amis_admin's is based on amis-based component development, you can freely replace or add built-in amis components in many places. I hope you can read the [amis documentation](https://baidu.gitee.io/amis/zh-CN/components/page), to have certain understanding of amis core components.

## Custom Single Operation Action

### Example-2

```python
# creating common ajax action
class TestAction(admin.ModelAction):
    # configuration action basic information
    action = ActionType.Dialog(label='custom_common_processing_actions', dialog=Dialog())

    # handling the action 
    async def handle(self, request: Request, item_id: List[str], data: Optional[BaseModel], **kwargs):
        # getting a list of data selected by a user from the database
        items = await self.fetch_item_scalars(item_id)
        # implement the action handling
        ...
        # returning the action handling result
        return BaseApiOut(data=dict(item_id=item_id, data=data, items=list(items)))


@site.register_admin
class ArticleAdmin(admin.ModelAdmin):
    page_schema = PageSchema(label='page_schema', icon='fa fa-file')
    model = Article
    # adding custom single item and bulk operation actions
    admin_action_maker = [
        lambda admin: TestAction(admin, name='test_action',flags=['item','bulk'])
    ]
```

Example-2 summary:

- defined a basic model action class 'TestAction', it's core is the 'handle' method.
  For details follow: [ModelAction](/amis_admin/ModelAction/#baseformadmin)

- through the `admin_action_maker` field, instantiated 'TestAction' class and bound single item and bulk operation actions to the current model management class


## Custom Bulk Operations Actions

### Example-3

```python
from fastapi_amis_admin import admin


# creating ajax form action
class TestFormAction(admin.ModelAction):
    # basic informaition about action configuration
    action = ActionType.Dialog(label='custom_form_action', dialog=Dialog())

    # creating an action form data model
    class schema(BaseModel):
        username: str = Field(..., title='username')
        password: str = Field(..., title='password', amis_form_item='input-password')
        is_active: bool = Field(True, title='is_active')

    # handling the action
    async def handle(self, request: Request, item_id: List[str], data: schema, **kwargs):
        # getting list of data selected by a user from the database
        items = await self.fetch_item_scalars(item_id)
        # implement the action handling
        ...
        # returning the action handling result
        return BaseApiOut(data=dict(item_id=item_id, data=data, items=list(items)))


@site.register_admin
class ArticleAdmin(admin.ModelAdmin):
    page_schema = PageSchema(label='page_schema', icon='fa fa-file')
    model = Article

    # adding custom single item and bulk operation actions
    admin_action_maker = [
        lambda admin: TestAction(admin, name='test_action',flags=['item','bulk'])
    ]

```

Example-3 is very similar to Example-2, but it allows to add a custom form, which is useful in many cases.

`schema` definition and usage are similar to `FormAdmin`.

## More about usage

Follow[demo program](https://github.com/amisadmin/fastapi_amis_admin_demo), or read related documentation below, it should be helpful.

- [ModelAdmin](/amis_admin/ModelAdmin/)

- [ModelAction](/amis_admin/ModelAction/#baseformadmin)

- [Action action button](https://baidu.gitee.io/amis/zh-CN/components/action?page=1)

