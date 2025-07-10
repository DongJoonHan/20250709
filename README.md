# 20250709

# 다운로드 위치
- Jenkins: https://www.jenkins.io/download/  에서 LTS Windows 버전
- OepnJDK: https://github.com/ojdkbuild/ojdkbuild  에서 17버전

# CPPCheck
## 설치위치

## 실행 명령
```sh
cppcheck  --enable=all --inconclusive --xml --xml-version=2 src 2> cppcheck.xml
```
```sh
"c:\Program Files\Cppcheck\cppcheck.exe"  --enable=all --inconclusive --xml --xml-version=2 src 2> cppcheck.xml
```


## MISRA 분석을 위한 실행
### Misra_2012.txt 파일 복사
- 위치: C:\TestTools

### 설정 파일 생성: misra.json
- 위치: C:\TestTools
- 파일 구성
```json
{
    "script": "misra.py",
    "args": [
        "--rule-texts=C:\\TestTools\\Misra_2012.txt"
    ]
}
```

### 실행명령
```sh
cppcheck.exe --addon="C:\TestTools\misra.json"  --xml --xml-version=2 . 2> cppcheck.xml
```


# CPD
## 다운로드
https://github.com/pmd/pmd/releases/tag/pmd_releases%2F6.55.0
에서, https://github.com/pmd/pmd/releases/download/pmd_releases%2F6.55.0/pmd-bin-6.55.0.zip 다운로드

## 실행
- Jenkins 용
```sh
C:\tools\pmd\bin\cpd --minimum-tokens 100 --files ./src --language cpp --format xml > cpd.xml || exit 0
```

- CSV 출력 (엑셀 확인용)
```sh
"C:\TestTools\pmd-bin-6.55.0\bin\cpd.bat" --minimum-tokens 100 --files . --language cpp --format csv > result.csv
```


# Lizard 순환 복잡도 분석
## 설치 주의 사항
CMD(명령 프롬프트)를 관리자 권한으로 실행한 후, pip install lizard 실행

## 실행 방법
- CSV 출력
```sh
"C:\Users\User\AppData\Local\Programs\Python\Python313\Scripts\lizard.exe" . --csv > lizard.csv
```

- Jenkins용 xml 출력
```sh
"C:\Users\User\AppData\Local\Programs\Python\Python313\Scripts\lizard.exe" . --xml > lizard.xml
```

## CSV 출력했을 때 각 열 정보
![image](https://github.com/user-attachments/assets/d660f376-6cf2-4dd2-a819-f0bc24424bea)




# CLOC (라인수, 주석라인수 확인)
## 다운로드
https://github.com/AlDanial/cloc

## 사용법
```
cloc.exe .
```

## Jenkins 플러그인
SLOCCount Plug-in: https://plugins.jenkins.io/sloccount/

# Doxygen
## Jenkins에서 Doxygen HTML Report 정상 출력하기
Jenkins 관리 -> Script Console 이동 후, 다음 실행

```
System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "")
```

# Node 연결
## 접속이 안되는 경우
다음과 같이 Jenkins 관리 -> Security 에서 Agent 정보 변경
![image](https://github.com/user-attachments/assets/00771fc1-de9e-4cb0-809f-0d4ca672632c)


## Windows 시작 시 연결을 위한 bat 파일 생성
가정: runNode.bat 이라 가정
```sh
java -jar C:\jenkins\agent.jar -url http://172.16.2.155/ -secret eebeda790a91f946c5c496a4bb7918ee0d4164dfe69dc48fe8ebd1160a475b7c -name "A_win10" -workDir "c:\jenkins"
```

## 시작프로그램 등록
1. 시작+R 키로 "실행"을 실행
2. shell:startup 입력해서 실행
3. 시작 프로그램에 runNode.bat 복사

# Virtual Box CLI 명령
## 주의 사항
Jenkins가 서비스로 실행되는 경우, 로그인 사용자가 VirtualBox 설치 사용자와 다르면 VM 실행이 안됨.
Jenkins의 서비스 실행을 VirtualBox 설치 사용자와 동일하게 변경 필요
- 단, 사용자 Password가 필요함 (없는 경우 생성 필요)
### 방법
Service -> Jenkins -> 로그온 탭 -> 사용자 지정 선택 -> 사용자를 현재 사용자로 설정
![image](https://github.com/user-attachments/assets/ec6956c8-fb8e-4f2d-bdff-0760d118d78a)


## win10Pure VM을 OfficialBuild 스냅샷으로 복원
```
VBoxManage snapshot "win10Pure" restore "OfficialBuild"
```
```
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" snapshot "win10Pure" restore "OfficialBuild"
```

## win10Pure VM 시작
```
VBoxManage startvm "win10Pure"
```
```
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" startvm "win10Pure"
```

## win10Pure VM 시작 (GUI 없이 백그라운드로 시작)
```
VBoxManage startvm "win10Pure" --type=headless
```

# Node_DX12 Job에서 자동으로 Windows 종료 명령
```
shutdown /s /t 60
```
/s: 종료 (/r은 재부팅)
/t 초: 설정 시간(초) 이후에 종료

# Pipeline
```
pipeline {
    agent any 
    
    environment {
        MSBUILD_PATH = 'C://Program Files (x86)//Microsoft Visual Studio//2022//Community//MSBuild//Current//Bin//MSBuild.exe'  
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [],
                    submoduleCfg: [],
                    userRemoteConfigs: [[url: 'https://github.com/microsoft/DirectXTK12.git']]
                ])
            }
        }

        stage('Build') {
            steps {
                bat "\"${env.MSBUILD_PATH}\" DirectXTK_Desktop_2022_Win10.vcxproj"  
            }
        }
        
        stage('Verification') {
            steps {
                bat "\"C://Program Files//Cppcheck//cppcheck.exe\"  --enable=all --inconclusive --xml --xml-version=2 src 2> cppcheck.xml"
                recordIssues enabledForFailure: true, aggregatingResults: true, tool: cppCheck(pattern: 'cppcheck.xml')
            }
        }
    }
}
```
# Gilab에서 연동
## Jenkins의 스크립트 빌드 명령
```
http://admin:API토큰명@20.39.197.62/job/JUnit4_Gitlab/build?token=12345
```

<img width="845" alt="image" src="https://github.com/user-attachments/assets/593e9065-2739-4a36-a66a-daed92873d2b">

## Gitlab 설정
<img width="952" alt="image" src="https://github.com/user-attachments/assets/e5aad143-29d5-4521-b211-e757eb79c31a">

<img width="752" alt="image" src="https://github.com/user-attachments/assets/73393263-f7e4-4114-aa09-118faeb53ef0">

