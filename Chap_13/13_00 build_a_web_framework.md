# Build a Web Framework

이전 12장에서는 Go언어를 사용하여 Web 응용 프로그램을 개발하는 방법에 대해 소개했습니다.    
기초적인 지식, 개발 도구 및 개발 기술을 소개했기 때문에 이 번장에서는 이러한 지식을 통해서     
간단한 Web 프레임 워크를 구현하도록 하겠습니다.    

먼저 Go 언어를 사용해서 완벽한 웹 프레임워크를 설계 합니다.  이 프레임워크는 주로 1 장에서   
소개 할  Web 프레임워크의 구조 규칙을 포함 합니다.  예를 들어, MVC 모드를 사용하여 개발을 할 경우   
프로그램 실행 프로세스 설계 등의  내용이 포함 됩니다. 2 장에서 소개 할 프레임워크의 첫 번째 기능인  라우팅,  
사용자가 접근한  URL에 기반하여  대응 처리 로직을 작성 하거나, 3 장에서 소개 할  처리 로직으로,    
공용으로 사용할  controller를 설계하거나 객체를 상속 한 후 처리 함수에서 어떻게 `response`와 `request`를    
처리하는지를 설명합니다. 4 장에서는 프레임워크의 일부 보조 기능을 설명합니다.     
예를 들어 로그처리에  대한 설정 및 정보 등을 설명합니다. 5 장에서는 Web 프레임워크를 기반으로 블로그를    
구현하는 방법에 대해 설명합니다. 여기에는 블로그 게시물, 수정, 삭제, 목록보기 등의 작업을 포함 합니다. 

 이 장을 통해서 개발자에게 어떻게 Web 응용 프로그램을 개발하는 방법과, 자신만의 디렉토리 구조를 만들거나    
 라우팅을 구현하거나, MVC 모드를 사용하는 법 등의 웹  개발의 전반적인 부분에 대한 설명합니다.    

프레임워크가 대세인 요즘, MVC 설계 방식은  더 이상 신비한 존재가 아닙니다.  프로그래머는  어떤 프레임 워크가 좋은지,    
어떤 점이 다른 프레임워크 보다 나은 점이 많은지와 같은 것이 토론 대상일 뿐입니다. 

프레임워크는 단지 도구일 뿐입니다. 원래 좋은 것도 나쁜 것도 없습니다.  단지 자신에게 가장  적절한 도구인가만   
중요할 뿐입니다.  자신의 용도에  맞으면 그만이므로, 개발자가 자신의 프레임워크를 작성할 수 있다면,     
다른 요구 사항에도 언제나 직접 개발할 수 있는 능력이 배양되게 됩니다. 

 


