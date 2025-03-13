Directory Traversal
공격자가 응용 프로그램을 실행 중인 서버에서 임의의 파일을 읽을 수 있도록 허용하는 웹 보안 취약점이다.

directory traversal공격을 통하여 애플리케이션 코드 및 데이터, 백앤드 시스템의 자격 증명 및 중요한 파일을 읽거나 접근할 수 있다. 또한 임의 파일에 쓰면서 데이터를 수정하여 서버를 제어할 수 있다.

Background
절대경로: / -> 최상위 디렉터리를 기준으로 찾음

상대경로:./ -> 현재 디렉터리를 기준으로 찾음

상위 경로이동: ../ -> 현재 디렉터리의 상위 디렉터리로 이동

example
<img src="/loadImage?filename=218.png">
loadImage url은 filename 매개변수를 사용하여 지정된 파일의 내용을 가져온다. 이미지 파일은 /var/www/images에 저장된다.

이미지를 load하기 위해 애플리케이션은 요청된 파일을 기본 디렉터리에 추가하고 파일 시스템 API를 사용하여 파일의 내용을 읽는다. 

/var/www/images/218.png
filename으로 요청된 파일의 경로는 위와 같다. 사용자의 입력을 바탕으로 파일을 load 할 때 path traversal을 이용하여 임의의 파일에 접근할 수 있다.

/var/www/images/../../../etc/passwd
filename=../../../etc/passwd를 입력하면 경로는 위와 같다. 그러면 /etc/passwd에 접근하면서 해당 파일의 내용이 출력된다.

실습을 통해 확인해보자.
         
           
해당 web site가 있다.
           
          
해당 image의 경로가 /image/filename=30.jpg이다.            
                  
         
URL을 보면 /image/filename=30.jpg이다. filename을 매개변수로 해당 경로의 파일을 load 하고 있다.        
          
           
그래서 filename=../../../etc/passwd를 입력하면 해당 파일의 내용을 읽을 수 있다.      
             
path traversal을 막는 방법은 .와 /를 filtering 하면 된다.          
          
우리는 path traversal filtering을 우회하는 방법을 같이 알아보자.      
그래서 filename=../../../etc/passwd를 입력하면 해당 파일의 내용을 읽을 수 있다.         
         
path traversal을 막는 방법은 .와 /를 filtering 하면 된다.          
         
우리는 path traversal filtering을 우회하는 방법을 같이 알아보자.          
           
Absolute path          
filename=/etc/passwd          
filename에 절대 경로를 사용하여 ../와 같은 상위 디렉터리를 거치지 않고 바로 실행시킬 수 있다.         
                
replace        
str_replace('../', '', filename)            
filename의 ../를 공백으로 처리하는 경우         
            
filename=....//....//....//etc/passwd           
../ 내부에 ../를 넣음으로써 ../를 만들어 우회할 수 있다.          
              
encoding        
filename=..%2F..%2F..%2Fetc%2Fpasswd           
filename=%2E%2E%2F%2E%2E%2F%2E%2E%2Fetc%2Fpasswd           
filename=..%252F..%252F..%252Fetc%252Fpasswd        
filename=%252E%252E%252F%252E%252E%252F%252E%252E%252Fetc%252Fpasswd          
          
사용자의 입력값을 보내기 전에 ../을 filtering 한다면 encoding이나 double encoding을 통하여 우회할 수 있다.         
            
base Folder          
filename=/var/www/images/1.jpg          
이렇게 filename에 file을 찾을 경로가 포함되어 있어야 한다면          
      
filename=/var/www/images/../../../etc/passwd           
해당 경로를 입력하고 path traversal을 해주면 우회할 수 있다.       
         
vaild file extension
file을 찾을 때 확장자 검증에 대한 필터링이 걸려 있다면 null byte를 이용하여 우회할 수 있다.      
         
filename=../../../etc/passwd%00.jpg         
이렇게 null byte를 사용하면 입력값은 null byte 전까지 들어간다. 하지만 확장자는 .jpg가 들어가기 때문에 확장자 검증을 우회할 수 있다.         

