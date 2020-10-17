# FileDescriptor
#TIL/concept

---

## File 이란?

- File
	- Linux 커널에서의 정의는, 추상체인 인터페이스인데, Read와 Write를 지원(구현)하는 모든 것을 총칭한다.
	- 네트워크 endpoint에도 파일이 생긴다고 할 수 있다.
	- 하드에 올라가는 것도, 메모리에 올라가는 것도, 소켓 통신의 소켓도 파일이라 할 수 있다.

## File Descriptor

File Identifier와 같은 의미라고 볼 수 있다.  
어떤 프로세스가 File에 접근할 때 사용하는 것이 File Descriptor이다.  
0부터 시작해서 정수값을 하나씩 부여받는데, 0, 1, 2 세 개의 값은 다음과 같이 특수한 의미로 지정되어 있다.  

- 0
	- stdin. 표준 입력
- 1
	- stdout. 표준 출력
- 2
	- stderr. 표준 에러

`/proc` 에 가면 여러 프로세스의 정보들을 확인할 수 있다. (process)  
그 중에는 `fd` 라는 폴더도 있다.  
해당 폴더 안에 0, 1, 2를 포함한 현재 프로세스에 할당된 file들의 번호를 볼 수 있다.  

일반 애플리케이션에서 네트워크가 밀리는 경우, (예를 들어 DB response가 늦어서 DB Connection Pool에 있는 모든 Connection을 물고 있는 경우), 네트워크 I/O도 파일이기 때문에 fd가 급증하게 된다.  
`ulimit -a` 로 확인하면 한 프로세스에서 열 수 있는 file의 최대 개수는 1024개임을 볼 수 있는데, 이 값을 늘리거나 하는 튜닝 포인트가 있을 수 있다.  
