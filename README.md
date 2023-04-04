# LightGBM 문서 번역

[LightGBM](https://github.com/microsoft/LightGBM)의 온라인 문서 저장소로 가짜연구소 6기 프로젝트입니다. 자세한 내용은 [LightGBM 온라인 문서 번역](https://chanrankim.notion.site/LightGBM-e981a2c31eaa460ab0b642aa840778a3) 팀 페이지를 참고하세요.

## 용어 번역 관리 시트

용어집은 편리성 때문에 구글 시트로 관리합니다. 자유롭게 추가하고 의견 주세요.

https://docs.google.com/spreadsheets/d/1Fz3Z_T4rTqxtZCRhh2ykgg2D4POpATXSnG22sZWzP0U/edit?usp=sharing

## 번역시 유의 사항

토론 https://github.com/Pseudo-Lab/lightgbm-docs/discussions/20 참조

## 번역 저장소 운영 방법

1. lightgbm-docs 저장소를 클론합니다. 작업할 LightGBM 버전에 해당하는 폴더를 rst 폴더 아래 생성합니다.
```sh
mkdir rst/v3.3.5
```
2. LightGBM 저장소를 클론 후 작업할 버전의 태그로 체크아웃합니다.
```sh
git checkout v3.3.5
```
3. `LightGBM/docs` 디렉토리를 `lightgbm-docs/rst/v3.3.5`로 복사합니다.
4. 작업자는 lightgbm-docs 저장소를 포크하여 `lightgbm-docs/rst/v3.3.5` 안의 파일을 번역하고 PR을 보냅니다.
5. `lightgbm-docs/rst/v3.3.5` 폴더에 있는 모든 문서의 번역이 완료되면 `lightgbm-docs/rst/v3.3.5` 안의 파일을 `LightGBM/docs`로 복사합니다. 그다음 `LightGBM/docs` 디렉토리에서 다음 명령으로 html 문서를 빌드합니다.
```sh
pip install sphinx 'sphinx_rtd_theme>=0.5'
export C_API=NO || set C_API=NO
make html
```
6. 빌드가 끝나면 `LightGBM/docs/_build/html` 폴더 안의 모든 파일과 폴더를 `lightgbm-docs/docs/v3.3.5`로 복사합니다.
7. `lightgbm-docs/docs/v3.3.5` 폴더를 깃허브에 푸시합니다.
8. `lightgbm-docs/docs/index.html` 페이지의 리다이렉션을 `lightgbm-docs/docs/v3.3.5` 폴더로 바꿉니다.
9. 새 버전(예를 들어 v3.3.6)이 나오면 `LightGBM/docs` 디렉토리의 버전간 diff를 `lightgbm-docs/rst/v3.3.5` 폴더에 반영하여 `lightgbm-docs/rst/v3.3.6` 폴더를 만듭니다. 그다음은 5번부터 반복합니다.
