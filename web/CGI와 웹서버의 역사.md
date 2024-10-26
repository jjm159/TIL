# CGI와 웹의 역사


### CGI가 표준출력으로 출력했을 뿐인데 어떻게 웹서버가 입력으로 받지?
Before the process that runs the CGI program is loaded, a Linux dup2 function
is used to redirect standard output to the connected descriptor that is associated
with the client. Thus, anything that the CGI program writes to standard output
goes directly to the client.