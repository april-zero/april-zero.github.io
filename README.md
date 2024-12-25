### Nginx 配置静态博客展示及文件下载

#### 1. 准备静态博客文件
1. 将静态博客内容存放在服务器指定目录（如 `/var/www/static_blog`）。
2. 设置权限以确保 Nginx 可读取静态文件：
   ```bash
   sudo chown -R www-data:www-data /var/www/static_blog
   sudo chmod -R 755 /var/www/static_blog
   ```

#### 2. 编辑 Nginx 配置文件
配置文件通常位于 `/etc/nginx/nginx.conf` 或 `/etc/nginx/sites-available/default`。

**配置模板：**
```nginx
server {
    listen 80;
    server_name yourdomain.com;

    # 指定静态博客文件的根目录
    root /var/www/static_blog;
    index index.html;

    # 为下载设置一个特殊路径
    location /download {
        alias /var/www/static_blog; # 文件实际路径
        autoindex on;               # 自动生成目录列表
        autoindex_exact_size off;   # 显示文件大小（可选）
        autoindex_localtime on;     # 显示本地时间
    }

    # 处理博客展示
    location / {
        try_files $uri $uri/ =404;
    }

    # 日志配置（可选）
    error_log /var/log/nginx/blog_error.log;
    access_log /var/log/nginx/blog_access.log;
}
```

#### 3. 启用并测试配置
1. 测试 Nginx 配置语法是否正确：
   ```bash
   sudo nginx -t
   ```
2. 重新加载 Nginx 服务：
   ```bash
   sudo systemctl reload nginx
   ```

#### 4. 验证访问
1. **博客展示**：在浏览器访问 `http://yourdomain.com`，查看静态博客页面。
2. **文件下载**：访问 `http://yourdomain.com/download`，浏览和下载博客文件。

#### 5. 其他优化配置

##### 5.1 安全性配置（限制下载页面访问）
为 `/download` 路径启用基本认证：
```nginx
location /download {
    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/.htpasswd;
    alias /var/www/static_blog;
    autoindex on;
}
```
创建 `.htpasswd` 文件：
```bash
sudo apt install apache2-utils  # 如果未安装 htpasswd 工具
htpasswd -c /etc/nginx/.htpasswd username
```

##### 5.2 性能优化（启用 gzip 压缩）
```nginx
gzip on;
gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
```

#### 总结
通过以上配置，静态博客可以正常展示，并提供特定路径实现文件下载。

