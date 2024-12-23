# tomcat과 servlet의 관계

## 톰캣이 뭐야?
- WAS이자 서블릿 컨테이너
- WAS?
	- `Web Application Server`. 웹 프로토콜을 사용하는 서버 프로그램
	- Tomcat은 웹 프로토콜을 사용하는 요청을 받아서 처리하고 반환해주는 WAS이고, 내부적으로 Servlet을 사용하여 요청을 처리
	- 웹 통신 프로토콜인 HTTP 요청을 받아서 서블릿이 처리 가능한 형태로 가공하고, 서블릿이 처리한 결과를 다시 HTTP 형식으로 만들어서 반환하는 게 톰캣의 역할
- 서블릿 컨테이너?
	- 서블릿의 생명주기를 관리, 서블릿을 사용해서 요청을 처리
	- 톰캣이 로드되면, 서블릿 클래스를 로드하고 서블릿 표준 인터페이스인 `init`을 호출해서 서블릿을 객체를 생성하여 가지고 있음
	- 이걸 사용해서 HTTP 요청을 처리하는 것
- `Java EE (Jakatra EE) 표준 스펙` 중 웹 애플리케이션 표준 스펙을 구현한 것이 Tomcat
	- Servlet, JSP, EJB, JPA, JMS, WebSocket 등 다양한 표준 존재
	- Tomcat은 이 중 웹 애플리케이션과 관련된 `Servlet`과 `JSP` 스펙을 구현한 구현체임

## 서블릿이 뭐야?
- __서버가 요청을 처리하고 응답을 반환하는 일련의 작업을 `추상화`한 것__
- cgi 같이 웹 서버와 상호작용을 위한 인터페이스. 여기서는 Tomcat이라는 WAS와 상호작용.
- 서블릿 인터페이스
	```java
	public interface Servlet {
		void init(ServletConfig config) throws ServletException;
		void service(ServletRequest req, ServletResponse res) throws ServletException, IOException;
		void destroy();
		ServletConfig getServletConfig();
		String getServletInfo();
	}
	```
	- (서블릿 java 오라클 공식 문서)[https://docs.oracle.com/javaee/7/api/javax/servlet/Servlet.html]
- java 웹 애플리케이션은 Servlet을 상속받아서 요청을 처리하도록 로직을 구현 
- Servlet class는 Servlet를 상속받아서 요청 처리 후 응답 반환
	- 실제 사용시에는 Servlet을 상속 받은 HttpServlet class를 상속 받아서 doGet, doPost, doDelete, ... 로 구현
	```java
	@WebServlet("/hello")
	public class HelloServlet extends HttpServlet {
		@Override
		protected void doGet(HttpServletRequest req, HttpServletResponse res) throws IOException {
			// 요청 처리
			res.setContentType("text/plain");
			res.getWriter().write("Hello, Servlet!");
		}
	}
	```
- 이렇게 만들어진 class는 톰캣이 로드해서 사용함. 
	- __의존성 역전!!!!!__
		- 톰캣이라는 상위 모듈은 Servlet이라는 인터페이스 추상으로 의존
		- 서블릿이라는 구체는 Servlet 안터페이스 추상으로 기능 노출
- 참고로, SpringBoot는 이 서블릿 클래스 위에서 동작함!
	- DispatcherServelt 확인!! DispatcherServlet.java에서 직접 확인 가능
	```java
	public class DispatcherServlet extends FrameworkServlet {
		// ...
	}
	```
	- 참고로 doService랑 doDispach를 보면 springboot의 코어 로직을 볼 수 있다.

## 톰캣이 서블릿을 실행하여 HTTP 요청을 처리하는 방법
- 톰캣은 스레드풀에서 스레드를 하나 가져옴
	- 그래서 스프링부트에서 스레드풀 설정할 때 tomcat 어쩌고 하는 거임
- 요청 URL에 매핑된 서블릿 객체를 가져옴
	- 톰캣 프로그램 시작할 때, 서블릿 클래스를 로드해서 만들어둔(servlet 인터페이스 함수인 init을 호출해서) servlet 객체를 가져옴
- 웹 통신 프로토콜인 HTTP에 맞게 들어온 요청을 ServletRequest 형태로 변환
- 서블릿 객체의 service를 호출(servlet의 인터페이스인 service 함수)하여 servlet 객체가 요청을 처리하도록 함
- 서블릿이 반환한 응답 객체인 ServletResponse를 다시 HTTP 응답 형식으로 바꿔서 client에게 응답 전송

## Servlet 자바 프로그램은 Tomcat에 어떻게 연동되는거야?
- `WAR (Web Application Archive)`
	- 표준 Java EE에서 정의한 웹 애플리케이션 구조
	- Tomcat은 특정 디렉터리 구조를 제공하고, 이 구조에서 java application 파일들을 읽어옴
	- java application은 이 구조에 맞게 Tomcat에게 애플리케이션 파일들을 제공해야 함
	- 이 파일들의 압축 파일을 WAR(Web Application Archive)라고 함
	- WAR 구조
	```
	MyApp.war
	├── META-INF/            <-- 메타데이터 (MANIFEST.MF 등)
	│   └── MANIFEST.MF
	├── WEB-INF/             <-- 웹 애플리케이션 설정 및 클래스 파일
	│   ├── web.xml          <-- 배포 서술자 (필수는 아님, Spring Boot는 자동 설정)
	│   ├── classes/         <-- 컴파일된 클래스 파일 (.class)
	│   ├── lib/             <-- 의존성 JAR 파일
	│   └── web.xml
	└── static/              <-- 정적 리소스 (CSS, JS, 이미지 등)
	└── *.jsp                <-- JSP 파일 (선택 사항)
	```
	- Servlet은 WEB-INF의 classes에 존재해야 함
	- Tomcat은 이 디렉터리에서만 search해서 web.xml에 명시된 클래스들을 로드함
- Tomcat이 읽을 수 있도록 Servelt과 URL 등록하기
	- 1. __web.xml__
		```xml
		<!-- web.xml -->
		<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
				xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
				xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
									http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
				version="3.1">

			<!-- 서블릿 정의 -->
			<servlet>
				<servlet-name>HelloServlet</servlet-name>
				<servlet-class>com.example.HelloServlet</servlet-class>
			</servlet>

			<!-- 서블릿 매핑 -->
			<servlet-mapping>
				<servlet-name>HelloServlet</servlet-name>
				<url-pattern>/hello</url-pattern>
			</servlet-mapping>

		</web-app>

		```
		- Tomcat이 /hello로 들어오는 요청을 HelloServlet 클래스에 매핑하도록 설정
	- 2. __annotation 사용하기__
		- Servlet 3.0부터는 web.xml 대신 @WebServlet 애너테이션을 사용하여 서블릿을 직접 매핑할 수 있어.
		```java
		// HelloServlet.java
		package com.example;

		import java.io.IOException;
		import javax.servlet.ServletException;
		import javax.servlet.annotation.WebServlet;
		import javax.servlet.http.HttpServlet;
		import javax.servlet.http.HttpServletRequest;
		import javax.servlet.http.HttpServletResponse;

		@WebServlet("/hello")  // 애너테이션을 사용한 URL 매핑
		public class HelloServlet extends HttpServlet {
			
			@Override
			protected void doGet(HttpServletRequest request, HttpServletResponse response)
					throws ServletException, IOException {
				response.setContentType("text/html");
				response.getWriter().println("<h1>Hello, World!</h1>");
			}
		}
		```
		- doGet 메서드를 통해 /hello로 들어오는 GET 요청을 처리
		- 클라이언트가 /hello URL로 요청을 보내면 HelloServlet이 실행되고, “Hello, World!“라는 HTML 응답이 반환

## Servlet Filter
- servlet과 마찬가지로 Tomcat이 로드할 수 있도록 web.xml에 등록하거나 어노테이션을 사용해서 Filter를 상속받은 클래스를 등록해야 한다.
- Filter를 상속 받은 클래스를 어노테이션으로 톰캣 클래스 로더에 등록하기
```java
import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;

@WebFilter("/*") // 모든 요청에 필터 적용
public class LoggingFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) {
        System.out.println("Logging Filter Initialized");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        System.out.println("Request received at Logging Filter");
        
        // 요청을 다음 필터 또는 서블릿으로 전달
        chain.doFilter(request, response);

        System.out.println("Response processed at Logging Filter");
    }

    @Override
    public void destroy() {
        System.out.println("Logging Filter Destroyed");
    }
}
```

## 스프링 부트와 톰캣
- spring-boot-starter-web을 의존성으로 추가하면 내장 Tomcat이 자동으로 포함
- Jar를 만들면 Tomcat도 포함됨!!
- 스프링부트 애플리케이션이 실행되면 Tomcat 서버 인스턴스를 생성하고 시작
- DispatcherServelt 클래스를 톰캣의 클래스 로더에 올려 놓기 때문에 톰캣이 요청이 들어오면 DispatcherServlet을 실행하게 됨
- 스프링부트로 개발한 Controller들은 DispatcherServlet에서 실행됨
	- DispatcherServlet의 doService, doDispach 함수 참고

## 스프링 부트의 Jar 구조를 보면 WAR 구조를 따르지 않네?
- Jar 구조를 보면 Tomcat에서 인식하는 WAR 구조를 따르지 않고 있음
- 그런데 어떻게 Tomcat은 class들을 로드하는 걸까?
- Tomcat은 java의 클래스 로더 메커니즘에 따라 클래스를 로드하는데, 클래스 로더는 커스텀 클래스 로드 기능을 제공함
- Springboot Loader는 Tomcat에게 커스텀 클래스 로더를 제공하여 Jar 안에 있는 BOOT-INF/classes와 BOOT-INF/lib를 클래스 로드 경로로 제공함
