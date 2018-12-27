# PLUG-AND-PLAY PATTERN
A pattern to design plug-able components for python

## Ý tưởng bao trùm
Plug-and-play pattern giúp cho dev tạo ra các component có thể tái sử dụng cho các dự án sau này, mà không phải modify quá nhiều, chỉ cần register component là có thể go-live.

## Các thành phần chính
### Components:
Lớp cao nhất, coi như là một bộ thư viện độc lập, có thể lắp vào các project khác nhau mà gần như không phải thay đổi. Gồm:

- Model
- Controller/View
- Route (đối với design REST API)

### Model: Trái tim của component - Model là thứ tồn tại duy nhất, những thứ khác, có hay không, không quan trọng!
Là một class chứa data và logic của component, bao gồm các thông tin về model và các method của model.

**Data của model:** là các trường thông tin mà một object cần có

**Method của model:** là các phương thức thao tác với data của model, bao gồm cả validation, và thao tác với database, tính toán... mà vốn model được design ra để phục vụ những task này.

### Controller/View:
Là một class chứa các method khác để tương tác với model, trong tình huống các method của model chưa đáp ứng được nhu cầu customize của app, hoặc cần một class phụ để thực hiện các logic khác (thường thấy khi làm MethodView đối với REST API).

### Route:
Là một Class/function đăng ký route cho component, dùng khi dev web API. Dev có thể override Class/function này bằng cách gán thêm các route nếu cần.

## Stack công nghệ:
Dev có thể tự do lựa chọn các stack công nghệ mà mình thấy thoải mái. Vì tính plug-able cao, nên dev có thể tái sử dụng các Model. Controller/View và Route có thể phải implement lại. 

Tuy nhiên điều này vẫn hỗ trợ tính plug-able vì Component của một stack có thể tái sử dụng nếu project sau sử dụng cùng stack công nghệ với Component.

Lời khuyên để tăng tính plug-able của Component, là package các Component này và đẩy lên registry.

## Chi tiết về các thành phần của component
### Model:
Các method cơ bản phải có:

```
class BaseModel():

    #Define data của model tại đây, hoặc trong __init___
    field1 = <Object>
    field2 = <Object>

    def __init__(self,*args,**kwargs):
    #Nhận data input của model
    #và khởi tạo các data này cho model, ví dụ:

    self.data = data

    (async) def validateCreate(self):
    #validate các data theo logic của model với method create
        return <model instance>

    (async) def validateRead(self):
    #validate các data theo logic của model với method read
        return <model instance>

    (async) def validateUpdate(self):
    #validate các data theo logic của model với method update
        return <model instance>

    (async) def validateDelete(self):
    #validate các data theo logic của model với method delete
        return <model instance>

    (async) def create(self):
    #insert data của model lên database
        return <model instance>

    @staticmethod
    (async) def read(query,*args,**kwargs):
    #query data của model lên database
        return <model instance>

    @staticmethod
    (async) def update(query,*args,**kwargs):
    #update data trong model lên database
        return <model instance>

    @staticmethod
    (async) def delete(query,*args,**kwargs):
    #delete data của model trên database
        return <Boolean>
```

Validate method sẽ không được tự động gọi khi gọi các method CRUD. Dev phải gọi validate explicit trước khi gọi các method CRUD.

### Controller/View:
Các method cơ bản phải có nếu sử dụng cho REST API:

```
import BaseModel
import MethodView

class BaseView(MethodView):

    (async) def post(self,*args,**kwargs):
    #handle GET request
        return <Response>

    (async) def post(self,*args,**kwargs):
    #handle POST request
        return <Response>

    (async) def put(self,*args,**kwargs):
    #handle PUT request
        return <Response>

    (async) def delete(self,*args,**kwargs):
    #handle DELETE request
        return <Response>
```

### Route:

Các function cơ bản cần phải có của Route:

```
import BaseView

baseView = BaseView.as_view()

function registerRoute(app,url,*args,**kwargs):
    """
    Nhận web app object để đăng ký các route ứng với các view function
    Biến path nhằm khai báo route cho Controller/View để dễ xử lý trong nội bộ component mà không cần biết đầy đủ route của object app.
    """

    #Có thể khai báo theo kiểu MethodView
    #Khai báo cách này sẽ đáp ứng được GET,POST,PUT,DELETE
    # nhưng phải xử lý thêm ở trong các method của class BaseView
    # nếu như một request POST có thể dùng cho nhiều mục đích.

    app.add_url_rule(f"{url}/<path>", view_func=baseView, methods=["POST"])

    #Hoặc có thể khai báo với từng route, từng function, từng method nếu như đòi hỏi phải customize function

    app.add_url_rule(f"{url}/login", view_func=BaseView().login, methods=["POST"])
    app.add_url_rule(f"{url}/register, view_func=BaseView().register, methods=["POST"])
    app.add_url_rule(f"{url}/id/<username>", view_func=BaseView().getUser, methods=["GET"])
```
