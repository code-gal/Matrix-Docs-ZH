### 如何在没有服务器的情况下作为开发者测试 CAS

[django-mama-cas](https://github.com/jbittel/django-mama-cas) 项目是一个基于 Django 的易于运行的 CAS 实现。

#### 先决条件

1. 创建一个新的虚拟环境：`python3 -m venv <你的虚拟环境>`
2. 激活你的虚拟环境：`source /path/to/your/virtualenv/bin/activate`
3. 安装 Django 和 django-mama-cas：
   ```sh
   python -m pip install "django<3" "django-mama-cas==2.4.0"
   ```
4. 在当前目录下创建一个 Django 项目：
   ```sh
   django-admin startproject cas_test .
   ```
5. 按照 [安装指南](https://django-mama-cas.readthedocs.io/en/latest/installation.html#configuring) 配置 django-mama-cas
6. 设置 SQLite 数据库：`python manage.py migrate`
7. 创建一个用户：
   ```sh
   python manage.py createsuperuser
   ```
   1. 使用你想要的用户名和密码。
   2. 其他字段可以留空。
8. 使用内置的 Django 测试服务器在端口 8000 上提供 CAS 端点：
   ```sh
   python manage.py runserver
   ```

现在你应该已经配置好了一个 Django 项目来提供 CAS 认证，并创建了一个用户。

#### 配置 Synapse（和 Element）使用 CAS

1. 修改你的 `homeserver.yaml` 以启用 CAS 并指向你本地运行的 Django 测试服务器：
   ```yaml
   cas_config:
     enabled: true
     server_url: "http://localhost:8000"
     service_url: "http://localhost:8081"
     #displayname_attribute: name
     #required_attributes:
     #    name: value
   ```
2. 重启 Synapse。

注意，上述配置假设主服务器在端口 8081 上运行，CAS 服务器在端口 8000 上运行，均在 localhost 上。

#### 测试配置

然后在 Element 中：

1. 访问指向你主服务器的 Element 的登录页面。
2. 点击单点登录按钮。
3. 使用 `createsuperuser` 创建的凭据登录。
4. 你应该已经登录。

如果你想重复这个过程，你需要先手动注销：

1. 访问 `http://localhost:8000/admin/`
2. 点击右上角的“注销”。