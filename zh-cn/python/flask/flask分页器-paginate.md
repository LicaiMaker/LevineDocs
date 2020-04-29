> 当数据需要分页加载显示时，需要用到flask中的paginate

# 使用paginate
```python
pagination = Person.query.order_by('p_id').paginate(page=page, per_page=per_page)
```
page表示那一页，per_page表示每页多少条数据，那么返回的pagination对象里面有什么东西呢？
```
has_next
#True if a next page exists.

has_prev
#True if a previous page exists

items = None
#the items for the current page

iter_pages(left_edge=2, left_current=2, right_current=5, right_edge=2)
#Iterates over the page numbers in the pagination. The four parameters control the thresholds how many numbers should be produced from the sides. Skipped page numbers are represented as None.

next(error_out=False)
#Returns a Pagination object for the next page.

next_num
#Number of the next page

page = None
#the current page number (1 indexed)

pages
#The total number of pages

per_page = None
#the number of items to be displayed on a page.

prev(error_out=False)
#Returns a Pagination object for the previous page.

prev_num
#Number of the previous page.

query = None
#the unlimited query object that was used to create this pagination object.

total = None
#the total number of items matching the query


```
> 更多详解：[http://www.pythondoc.com/flask-sqlalchemy/api.html?highlight=pagination#flask.ext.sqlalchemy.Pagination](http://www.pythondoc.com/flask-sqlalchemy/api.html?highlight=pagination#flask.ext.sqlalchemy.Pagination)

# 创建视图并美化
```python
@main.route('/showPersonList', methods=['GET', 'POST'])
def show_content():
    page = request.args.get('page', 1, type=int)
    per_page = request.args.get('per_page', 3, type=int)
    pagination = Person.query.order_by('p_id').paginate(page=page, per_page=per_page)
    return render_template('content.html', pagination=pagination, per_page=per_page)

```
得到两个参数page和per_page，并进行分页处理
> 注：url_for的用法，如果视图函数里需要参数，则{{url_for('index.html',param=param)}},否则不需要后面的参数

> 思考：这里可以不采用分页器，也能达到分页的效果吗？能，使用limit和offset即可，打开paginate的源码就可以看到里面使用了这两个方法。

接下来编写一下HTML页面，展现分页效果
需要引入bootstrap，里面拥有好看的分页效果：
```html
{% extends 'bootstrap/base.html' %}

{% block content %}
    {% for p in pagination.items %}
        <ul>{{ p.p_name }}</ul>
    {% endfor %}
    <nav aria-label="Page navigation">
        <ul class="pagination">
            {% if pagination.has_prev %}
                <li>
                    <a href="{{ url_for('main.show_content') }}?page={{ pagination.prev_num }}&per_page={{ per_page }}" aria-label="Previous">
                        <span aria-hidden="true">&laquo;</span>
                    </a>
                </li>
            {% else %}
                <li class="disabled" >
                    <a href="#" aria-label="Previous">
                        <span aria-hidden="true">&laquo;</span>
                    </a>
                </li>

            {% endif %}

            {% for page in range(1,pagination.pages + 1) %}
                {% if page %}
                    {% if page != pagination.page %}
                        <li><a href="{{ url_for('main.show_content')}}?page={{ page }}&per_page={{ per_page }}">{{ page }}</a></li>
                    {% else %}
                        <li class="active"><a href="{{ url_for('main.show_content') }}?page={{ page }}&per_page={{ per_page }}">{{ page }}</a></li>
                    {% endif %}

                {% endif %}
            {% endfor %}

            {% if pagination.has_next %}
                 <li>
                    <a href="{{ url_for('main.show_content') }}?page={{ pagination.next_num }}&per_page={{ per_page }}" aria-label="Next">
                        <span aria-hidden="true">&raquo;</span>
                    </a>
                </li>
            {% else %}
                <li class="disabled" >
                    <a href="#" aria-label="Next">
                        <span aria-hidden="true">&raquo;</span>
                    </a>
                </li>
            {% endif %}
        </ul>
    </nav>

{% endblock %}
```

> 注：上面的` {% for page in range(1,pagination.pages + 1) %}`也可以换成`{% for page in pagination.iter_pages() %}`