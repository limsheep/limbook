参考：https://segmentfault.com/a/1190000022109061

使用HtmlUnit

TestBrowser:

```java
import com.gargoylesoftware.htmlunit.BrowserVersion;
import com.gargoylesoftware.htmlunit.NicelyResynchronizingAjaxController;
import com.gargoylesoftware.htmlunit.SilentCssErrorHandler;
import com.gargoylesoftware.htmlunit.WebClient;
import com.gargoylesoftware.htmlunit.html.*;

import java.io.IOException;

/**
 * 功能描述：
 *
 * @author Limy
 * @since 2021/6/21 0:57
 */
public class TestBrowser {
    public static void main(String[] args) {
        try(WebClient webClient = new WebClient(BrowserVersion.CHROME)) {
            /******配置webClient******/
            //ajax
            webClient.setAjaxController(new NicelyResynchronizingAjaxController());
            //支持js
            webClient.getOptions().setJavaScriptEnabled(true);
            //忽略js错误
            webClient.getOptions().setThrowExceptionOnScriptError(false);
            //忽略css错误
            webClient.setCssErrorHandler(new SilentCssErrorHandler());
            //不执行CSS渲染
            webClient.getOptions().setCssEnabled(false);
            //超时时间
            webClient.getOptions().setTimeout(3000);
            //允许重定向
            webClient.getOptions().setRedirectEnabled(true);
            //允许cookie
            webClient.getCookieManager().setCookiesEnabled(true);

            // 开始请求网站
            HtmlPage mainPage = webClient.getPage("https://gitee.com/");
            HtmlPage loginPage = ((DomElement) mainPage.getByXPath("/html/body/header/div/div/div[2]/div/div[1]/div[2]/a[1]").get(0)).click();

            /*
             * 获取登陆表单，表单如果是依赖js或css生成的，要等待加载完成，现有框架里等待方法不完善
             * 这里可以采用循环等待的方案，等到全部资源加载完，获取到了要取的表单元素再继续执行
             */
            while (loginPage.getByXPath("/html/body/div[2]/div[2]/div/div[1]/div[2]/div[1]/form[1]").size() == 0) {
                Thread.sleep(500);
            }

            //获取登陆表单元素
            HtmlForm form = (HtmlForm) loginPage.getByXPath("/html/body/div[2]/div[2]/div/div[1]/div[2]/div/form[1]").get(0);
            //用户名input
            HtmlTextInput username = (HtmlTextInput) form.getElementsByAttribute("input", "id", "user_login").get(0);
            //密码input
            HtmlPasswordInput password = (HtmlPasswordInput) form.getElementsByAttribute("input", "id", "user_password").get(0);
            //设置input的value
            username.setValueAttribute("Yang");
            password.setValueAttribute("ktu4N6USrEuDPGV");
            //登陆
            HtmlPage home = ((DomElement) loginPage.getByXPath("/html/body/div[2]/div[2]/div/div[1]/div[2]/div[1]/form[1]/div[2]/div/div/div[4]/input").get(0)).click();

            //搜索项目
            HtmlPage searchPage = webClient.getPage("https://gitee.com/search?utf8=%E2%9C%93&type=&fork_filter=on&q=java");
            //打印列表
            HtmlElement a = (HtmlElement) searchPage.getByXPath("/html/body/div[3]/div[1]/div/div[2]").get(0);
            System.out.println(searchPage.getBaseURL());
            System.out.println(a.getTextContent());
        } catch (IOException | InterruptedException exception) {
            exception.printStackTrace();
        }
    }
}
```

