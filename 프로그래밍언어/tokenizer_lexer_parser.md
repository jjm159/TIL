
## tokenizer, lexer, parser

[Anatomy of a Compiler](http://www.cs.man.ac.uk/~pjj/farrell/comp3.html)

위 글에서도 그렇고 다른 자료를 봐도 그렇고 명확하게 이 세 개념을 구분짓지는 않는 것 같다. 그래도 여러 자료들을 읽어보고 나름대로 종합적인 판단을 해보았다. 

### tokenizer : 입력된 데이터를 '의미 단위'로 쪼갬

### lexer : 입력된 데이터의 의미를 분석함 ( 진짜 토큰은 여기서 만들어진다 )

→ 토큰은 '의미'를 가진 최소 단위다. tokenizer는 아직 그 의미를 부여하지 못했다. 쪼갰을 뿐 부여를 못했음. 이걸 lexer가 해주는 것. 

### parser : 입력된 데이터를 가공하여  원하는 포맷으로 뽑아줌

→ 원하는 포맷이란, JSON Parser에게는 JOSN 포맷일 테고, 컴파일러에게는 Parse Tree를 말하는 것일 테고.

위 글에서 얘기하기를 a Tokenizer, or as it is sometimes called, a Lexer. 라고 한다. 아래 그림에서 보이듯, lexer의 모습은 보이지도 않는다. 그래도 Tokenizer가 lexer라고도 불린다는 것을 보면, Tokenizer와 lexer의 일이 비슷하다고 생각해볼 수 있다.

텍스트로 작성된 코드를 Parse Tree로 바꾸는 것. 이것이 컴파일러의 Front End에서 할 일이다. Paser는 Back end에서 사용한 Parse Tree를 만드는 역할을 한다. 그리고 이렇게 트리를 만들기 위해 먼저 소스 코드를 트리로 만들기 쉽게 만들어줘야 한다. 이게 tokenizer가 할 일이다. 

lexer는 '어휘 분석기'다. tokenizer는 '토큰 만들기'다. token은 a thing serving as a visible or tangible representation of a fact, quality, feeling 라는 뜻에서 알 수 있듯, 그냥 조각이 아니다. 어떤 의미를 가지는 조각이다. tokenizer는 소스코드로 부터 의미를 가지는 토큰을 만드는 역할을 한다. '의미'는 lexer가 분석한다. 이것이 소스트리의 요소가 될 자질이 있는지 검사하는 것이다.
