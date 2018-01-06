```
mkdir github
cd github
git clone git://git.openssl.org/openssl.git
cd openssl
# Prepare openssl source, we use 1.0.2n stabs version here
git checkout -b 1.0.2n tags/OpenSSL_1_0_2n
./Configure darwin64-x86_64-cc -shared
make
```

```
git clone git@github.com:aleecoin/nginx.git
cd nginx
git checkout branches/stable-1.12-macosx
```

```
brew install automake berkeley-db4 libtool boost --c++11 miniupnpc openssl pkg-config protobuf qt libevent pcre libxslt libxml2 gd geoip gperftools

brew link --force libxml2
brew link --force libxslt
```

# Not necessary if you are using the forked branch branches/stable-1.12-macosx
# Manually path auto/lib/libxslt/conf to fix libxml2 and libxslt -L and -I, e.g.

```
diff --git a/auto/lib/libxslt/conf b/auto/lib/libxslt/conf
index 3a0f37be..3d592c12 100644
--- a/auto/lib/libxslt/conf
+++ b/auto/lib/libxslt/conf
@@ -156,6 +156,23 @@ if [ $ngx_found = no ]; then
 fi


+if [ $ngx_found = no ]; then
+
+    # Homebrew
+
+    ngx_feature="libexslt in /usr/local/"
+    ngx_feature_path="/usr/local/include/libxml2/libxml /usr/local/include"
+
+    if [ $NGX_RPATH = YES ]; then
+        ngx_feature_libs="-R/usr/local/lib -L/usr/local/lib -lexslt"
+    else
+        ngx_feature_libs="-L/usr/local/lib -lexslt"
+    fi
+
+    . auto/feature
+fi
+
+
 if [ $ngx_found = yes ]; then
     if [ $USE_LIBXSLT = YES ]; then
         CORE_LIBS="$CORE_LIBS -lexslt"
```

# Manually patch auto/lib/openssl/conf

```
diff --git a/auto/lib/openssl/conf b/auto/lib/openssl/conf
index e7d3795b..567f7fda 100644
--- a/auto/lib/openssl/conf
+++ b/auto/lib/openssl/conf
@@ -36,10 +36,10 @@ if [ $OPENSSL != NONE ]; then
             have=NGX_OPENSSL . auto/have
             have=NGX_SSL . auto/have

-            CORE_INCS="$CORE_INCS $OPENSSL/.openssl/include"
-            CORE_DEPS="$CORE_DEPS $OPENSSL/.openssl/include/openssl/ssl.h"
-            CORE_LIBS="$CORE_LIBS $OPENSSL/.openssl/lib/libssl.a"
-            CORE_LIBS="$CORE_LIBS $OPENSSL/.openssl/lib/libcrypto.a"
+            CORE_INCS="$CORE_INCS $OPENSSL/openssl/include"
+            CORE_DEPS="$CORE_DEPS $OPENSSL/openssl/include/openssl/ssl.h"
+            CORE_LIBS="$CORE_LIBS $OPENSSL/openssl/libssl.a"
+            CORE_LIBS="$CORE_LIBS $OPENSSL/openssl/libcrypto.a"
             CORE_LIBS="$CORE_LIBS $NGX_LIBDL"

             if [ "$NGX_PLATFORM" = win32 ]; then
```

# Configure and Make

```
mkdir -p $HOME/nginx/tmp

./auto/configure --prefix=$HOME/nginx \
--sbin-path=$HOME/nginx/sbin/nginx \
--modules-path=$HOME/nginx/modules \
--conf-path=$HOME/nginx/etc/nginx.conf \
--error-log-path=$HOME/nginx/log/error.log \
--http-log-path=$HOME/nginx/log/access.log \
--http-client-body-temp-path=$HOME/nginx/tmp/client_body \
--http-proxy-temp-path=$HOME/nginx/tmp/proxy \
--http-fastcgi-temp-path=$HOME/nginx/tmp/fastcgi \
--http-uwsgi-temp-path=$HOME/nginx/tmp/uwsgi \
--http-scgi-temp-path=$HOME/nginx/tmp/scgi \
--pid-path=$HOME/nginx/run/nginx.pid \
--lock-path=$HOME/nginx/run/lock/subsys/nginx \
--user=nginx \
--group=nginx \
--with-http_ssl_module \
--with-http_v2_module \
--with-http_realip_module \
--with-http_addition_module \
--with-http_xslt_module=dynamic \
--with-http_image_filter_module=dynamic \
--with-http_geoip_module=dynamic \
--with-http_sub_module \
--with-http_dav_module \
--with-http_flv_module \
--with-http_mp4_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_random_index_module \
--with-http_secure_link_module \
--with-http_degradation_module \
--with-http_slice_module \
--with-http_stub_status_module \
--with-http_perl_module=dynamic \
--with-mail=dynamic \
--with-mail_ssl_module \
--with-pcre \
--with-pcre-jit \
--with-stream=dynamic \
--with-stream_ssl_module \
--with-google_perftools_module \
--with-debug \
--with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic' \
--with-openssl=/Users/banana/Documents/github

make

make install
```
