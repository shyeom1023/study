# 초기설정

[Chocolatey](https://chocolatey.org/)를 사용해 Dart SDK를 설치할 수 있습니다.

 **중요:** 이 명령들은 관리자 권한으로 실행해야 합니다. 다음은 관리자 권한으로 명령을 실행하는 방법입니다.

1. Windows+R을 눌러 **실행** 창을 엽니다.
2. 박스에 `cmd`를 입력합니다.
3. Ctrl+Shift+Enter를 누릅니다.

Dart SDK 설치:

```
C:\> choco install dart-sdk
```

Dart SDK 업데이트:

```
C:\> choco upgrade dart-sdk
```

SDK는 디폴트로 `C:\tools\dart-sdk`에 설치됩니다. [`ChocolateyToolsLocation`](https://stackoverflow.com/questions/19752533/how-do-i-set-chocolatey-to-install-applications-onto-another-drive/68314437#68314437) 환경 변수를 선택한 설치 디렉터리로 설정하여 설치 디렉터리를 변경할 수 있습니다.

설치 후 Dart SDK의 실행 파일을 사용할 수 없다면, SDK의 경로를 PATH에 추가하십시오:

1. Windows 검색을 열고, `env`을 입력합니다.
2. **시스템 환경 변수 편집**을 클릭합니다.
3. **환경 변수…**를 클릭합니다.
4. 사용자 변수 섹션에서, **PATH**를 선택하고 **편집…**을 클릭합니다.
5. **새로 만들기**를 클릭하고, `dart-sdk` 디렉터리의 경로를 입력합니다.
6. 열려 있는 각 창에서 **적용** 또는 **확인**을 클릭하고 팝업 창을 닫은 후 경로 수정을 적용합니다.