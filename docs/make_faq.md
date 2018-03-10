
LuaJIT
http://luajit.org/download/LuaJIT-2.0.5.tar.gz
make
make install PREFIX=/usr/local/luajit/



//先导入环境变量,告诉nginx去哪里找luajit
export LUAJIT_LIB=/usr/local/luajit/lib
export LUAJIT_INC=/usr/local/luajit/include/luajit-2.0


https://github.com/simplresty/ngx_devel_kit/tags

https://github.com/openresty/lua-nginx-module/tags


# --with-ld-opt
其中的 --with-ld-opt=-Wl,-rpath,/usr/local/luajit/lib:/usr/local/luajit/include/luajit-2.0 参数的意思是： 
这是链接器选项，目的是把 /usr/local/lib 和 /opt/openrersty/luajit/lib 这两个路径添加进 
nginx 可执行文件的 RPATH 头中，这样在启动的时候，系统就可以找到正确的动态链接库文件。