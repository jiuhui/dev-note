# favicon.ico

favicon.ico文件是浏览器收藏网址时显示的图标，当客户端使用浏览器页面时，浏览器会自己主动发起请求获取页面的favicion.ico文件。这样就会引起一些问题。解决办法。

- 前端页面添加以下代码不让浏览器主动请求。

  ```html
      <link rel="icon" href="/userLogin/css/favicon.ico" type="image/x-icon" />
  ```

- 后端代码通过filter拦截/favicon的请求返回图标

  ```java
  public class FaviconFilter extends OncePerRequestFilter {
  
      static final String FAVICON_PATH = "/favicon.ico";
  
      private final byte[] faviconFile;
  
      public FaviconFilter(byte[] faviconFile) {
          this.faviconFile = faviconFile;
      }
  
      @Override
      protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
          if (FAVICON_PATH.equals(request.getRequestURI())) {
              response.setStatus(HttpServletResponse.SC_OK);
              response.getOutputStream().write(faviconFile);
          } else {
              filterChain.doFilter(request, response);
          }
      }
  }
  ```

- nginx配置

  ```lua
  server {
      server_name www.mylinuxops.com;
      access_log /var/log/nginx/access.log access_json;
      location / {
          root /data/www;
          index index.html;
    }
      location = /favicon.ico {       #精确定位ifavicon.ico文件
          log_not_found off;          #找不到文件时日志不记录
          access_log off;             #关闭日志记录
   }
  }
  ```

  