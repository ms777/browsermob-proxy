# BrowserMob Proxy with Selenium

Due to some conflicts with different versions of the guava library, Selenium and BrowserMob have stopped working together.

This is a BrowserMob version, based on 2.1.6, which works with Selenium.

For details of the libraries used look into buildBrowsermob.xml

The only extern library required is 'selenium-server-standalone-3.14.0.jar'. Together with the 'browsermob-3.14.0.jar' from the releases provided here everything should work


Test Code:
```java
import java.io.File;
import java.io.IOException;

import org.openqa.selenium.NoSuchWindowException;
import org.openqa.selenium.Proxy;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.remote.CapabilityType;

import net.lightbody.bmp.BrowserMobProxy;
import net.lightbody.bmp.BrowserMobProxyServer;
import net.lightbody.bmp.client.ClientUtil;
import net.lightbody.bmp.core.har.Har;
import net.lightbody.bmp.proxy.CaptureType;

public class TestWithSelenium {

	public static void main(String[] args) throws InterruptedException, IOException  {
		// start the proxy
	    BrowserMobProxy proxy = new BrowserMobProxyServer();
	    proxy.setTrustAllServers(true);
	    proxy.start(0);
	    
	    // get the Selenium proxy object
	    Proxy seleniumProxy = ClientUtil.createSeleniumProxy(proxy);
	    System.out.println("proxy: " + seleniumProxy.toJson());
	    
		System.setProperty("webdriver.chrome.driver", "C:\\Users\\martin\\git\\libraries\\selenium\\chromedriver.exe");
		
		ChromeOptions  options = new ChromeOptions();
	    options.setCapability(CapabilityType.PROXY, seleniumProxy);
//	    options.setCapability(CapabilityType.ACCEPT_SSL_CERTS, true);
	    options.setCapability(CapabilityType.ACCEPT_INSECURE_CERTS, true);
	    options.addArguments("--ignore-certificate-errors");
	    options.addArguments("--user-data-dir=C:\\Users\\martin\\Downloads\\chrome");

	    WebDriver driver = new ChromeDriver(options);
	    
	    // enable more detailed HAR capture, if desired (see CaptureType for the complete list)
	    proxy.enableHarCaptureTypes(
	    		CaptureType.REQUEST_CONTENT, CaptureType.REQUEST_HEADERS, CaptureType.REQUEST_BINARY_CONTENT,
	    		CaptureType.RESPONSE_CONTENT, CaptureType.RESPONSE_HEADERS, CaptureType.RESPONSE_BINARY_CONTENT);

	    // create a new HAR with the label "yahoo.com"
	    proxy.newHar("yahoo.com");

	    // open yahoo.com
	    driver.get("https://de.yahoo.com");
	    try {
	    	while (1==1) {
//	    		System.out.println(driver.getWindowHandle());
	    		driver.getWindowHandle();
	    		Thread.sleep(500);
	    	}
	    } catch (NoSuchWindowException e) {
	    	System.out.println("NoSuchWindowException, terminating ...");
	    } finally {
	    	// get the HAR data
	    	Har har = proxy.getHar();
	    	har.writeTo(new File("C:\\Users\\martin\\Downloads\\eclipse.har"));
	    	proxy.stop();
	    	driver.quit();
	    }
	}
}
```
