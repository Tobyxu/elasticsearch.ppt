# ElasticSearch-head插件

![](/assets/42.png)


安装elasticsearch-head插件


因为head是一个用于管理Elasticsearch的web前端插件，该插件在es5版本以后采用独立服务的形式进行安装使用（之前的版本可以直接在es安装目录中直接安装），因此需要安装nodejs、npm

https://github.com/mobz/elasticsearch-head


yum -y install nodejs npm

cd elasticsearch-head/

npm install

配置插件

vi /etc/elasticsearch/elasticsearch.yml

加入配置:
```
http.cors.enabled: true
http.cors.allow-origin: "*"
```

修改Gruntfile.js文件

```
connect: {
            server: {
                options: {
                    /* 默认监控：127.0.0.1,修改为：0.0.0.0 */
                    hostname: '0.0.0.0',
                    port: 9100,
                    base: '.',
                    keepalive: true
                }
            }
```

修改head/_site/app.js

```
app.App = ui.AbstractWidget.extend({
        defaults: {
            base_uri: null
        },
        init: function(parent) {
            this._super();
            this.prefs = services.Preferences.instance();
            this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://192.168.40.4:9200";
            if( this.base_uri.charAt( this.base_uri.length - 1 ) !== "/" ) {
                // XHR request fails if the URL is not ending with a "/"
                this.base_uri += "/";
            }

...
```

启动插件（后台启动方式）
cd /usr/share/elasticsearch-head/node_modules/grunt/bin/
nohup ./grunt server & exit