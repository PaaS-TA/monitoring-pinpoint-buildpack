# PAAS-TA-PINPOINT-MONITORING-BUILDPACK

### PaaS-TA에 PinPoint 빌드팩 등록
PaaS-TA에서 PinPoint 빌드팩을 등록하기 위해서는 빌드팩 소스 Clone, 빌드팩 패키지, 업로드의 3가지 절차가 필요하다. 하단에 순서대로 설명을 기재한다.


#### 1. 빌드팩 소스 clone
git 명령어를 이용하여 소스를 clone 한다. git 명령어를 사용하기 위해서는 git의 설치가 요구된디. 사용자의 환경에 맞게 git을 설치한다.

`git clone https://github.com/PaaS-TA/PAAS-TA-PINPOINT-MONITORING-BUILDPACK.git`  


#### 2. 빌드팩 패키지
빌드팩 패키지는 clone한 소스를 바탕으로 빌드팩 패키지 파일을 생성하는 단계이다. 빌드팩을 패키지 하기 위해서 ruby와 bundler의 설치가 요구된다. 요구 버전은 Ruby 2.2.4, bundler 1.13.6 이다. 

`bundle install`  
```
...
Bundle complete! 11 Gemfile dependencies, 29 gems now installed.
Use `bundle show [gemname]` to see where a bundled gem is installed.
```  

`bundle exec rake package OFFLINE=true`  
※ 'OFFLINE=true' 옵션은 오프라인 빌드팩으로 패키징하는 옵션이다. 오프라인 빌드팩은 빌드팩이 동작하는데 필요한 모든 컴포넌트들을 다운로드하여 패키징에 포함시킨다. 오프라인 빌드팩을 사용할 경우, 오프라인 상태에서 애플리케이션 배포가 가능하다.

```
...
Creating build/java-buildpack-pinpoint-monitoring-121a22c.zip
```  

다음 경로에 패키징 파일이 생성된다.
`build/java-buildpack-pinpoint-monitoring-121a22c.zip`


#### 3. 빌드팩 업로드
패키징된 빌드팩을 PaaS-TA에 업로드한다. 이때 PaaS-TA는 빌드팩 업로드 권한을 가진 사용자로 로그인 할 것을 요구한다. 일반 사용자는 빌드팩을 업로드 할 수 없으므로 관리자 계정으로 로그인하여 빌드팩 업로드를 수행한다.

- 빌드팩 업로드  

빌드팩 업로드 명령어는 다음의 형태로 입력하게 되어있다.  
`cf create-buildpack <생성할 이름> <패키지 파일> <우선순위>`  

<생성할 이름> : PaaS-TA에서 사용하는 해당 빌드팩 고유의 이름을 입력한다.  
<패키지 파일> : 패키지된 파일의 경로와 파일명을 입력한다.  
<우선순위> : 검출(detect) 우선순위이다. 일반 Java 빌드팩보다 후순위로 우선순위를 지정한다.  

<div id='notice-01'></div>
※ 기본적으로 빌드팩은 검출 기준을 가지고 있다. 이를 통해 애플리케이션 배포시, 사용자가 빌드팩을 지정해주지 않아도 해당 소스에 맞는 빌드팩을 자동으로 찾아준다. 이떄, 같은 검출 기준을 가진 빌드팩이 여러개 있을 경우에는 우선순위에 따라 가장 우선순위가 높은 빌드팩을 사용하게 된다. 전자정부 프레임워크 빌드팩은 Java 빌드팩과 동일한 검출 기준을 갖고 있기 때문에 전자정부 프레임워크 빌드팩의 우선순위가 Java 빌드팩보다 높을 경우, 빌드팩을 지정하지 않고 배포하는 일반 Java 애플리케이션이 전자정부 프레임워크 빌드팩을 사용하게 된다. 이러한 혼란을 방지하기 위해 전자정부 프레임워크 빌드팩은 Java 빌드팩보다 우선 순위를 낮게 지정한다. (번호가 낮을 수록 우선순위가 높기 때문에 Java 빌드팩보다 높은 번호로 지정한다.)

`cf create-buildpack java_buildpack_pinpoint build/java-buildpack-pinpoint-monitoring-121a22c.zip 12`  

```
Creating buildpack java_buildpack_pinpoint...
OK

Uploading buildpack java_buildpack_pinpoint...
Done uploading
OK
```

- 업로드 된 빌드팩을 확인  

`cf buildpacks`  

```
buildpack                position   enabled   locked   filename
java_buildpack_offline   1          true      false    java-buildpack-offline-v3.10.zip
staticfile_buildpack     2          true      false    staticfile_buildpack-cached-v1.3.12.zip
java_buildpack           3          true      false    java-buildpack-v3.10.zip
ruby_buildpack           4          true      false    ruby_buildpack-cached-v1.6.27.zip
nodejs_buildpack         5          true      false    nodejs_buildpack-cached-v1.5.22.zip
go_buildpack             6          true      false    go_buildpack-cached-v1.7.14.zip
python_buildpack         7          true      false    python_buildpack-cached-v1.5.11.zip
php_buildpack            8          true      false    php_buildpack-cached-v4.3.21.zip
binary_buildpack         9          true      false    binary_buildpack-cached-v1.0.5.zip
dotnet_core_buildpack    10         true      false    dotnet-core_buildpack-cached-v1.0.4.zip
pinpoint_buildpack       11         true      false    java-buildpack-offline-pinpoint-v2.zip
java_buildpack_pinpoint   12         true      false    java-buildpack-pinpoint-monitoring-121a22c.zip
```
