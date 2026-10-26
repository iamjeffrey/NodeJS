# [Windows 11] Rust 기반의 초고속 Node.js 버전 관리 도구 `fnm` 설치 및 설정 가이드

---

## 1. `fnm(Fast Node Manager)` 설치 (PowerShell)

1. PowerShell을 열고 아래 명령어로 `fnm` 설치
   - https://winget.run/pkg/Schniz/fnm
   ```powershell
   winget install -e --id Schniz.fnm
   ```
2. 설치 확인
   ```powershell
   fnm -V
   ```

---

## 2. PowerShell 프로파일($PROFILE) 설정
터미널을 열 때마다 fnm 환경이 자동으로 연동되도록 설정합니다.

 - `$PROFILE`의 정확한 폴더 위치를 확인하고 싶다면 `$PROFILE`를 커맨드를 입력하면 전체 경로가 출력됩니다.

1. 메모장에서 프로파일 파일을 엽니다.
   ```powershell
   # Microsoft_PowerShell_profile.ps1 개인설정파일 메모장 열림 
   notepad $PROFILE
   ```

2. 파일 하단에 아래 코드를 추가하고 저장(`Ctrl + S`)합니다. 
   ```powershell
   # fnm 자동 환경 설정 (fnm 설치 전 Node 설치 된 경우, 기존 시스템 Node와 격리)
   # [참고] FNM_DIR 저장 위치 변경 안내 (기본값: AppData\Roaming\fnm)
   #       기본적으로 winget 설치 시 시스템이 경로를 자동 지정하므로 별도 설정이 필수 아님
   fnm env --use-on-cd --shell powershell | Out-String | Invoke-Expression

   # (선택) 기존 PowerShell 기본 별칭 제거 및 Bash 스타일 단축어 설정
   if (Get-Alias ls -ErrorAction SilentlyContinue) { Remove-Item alias:ls -Force }
   if (Get-Alias rm -ErrorAction SilentlyContinue) { Remove-Item alias:rm -Force }

   # 1. ls 함수
   function ls {
       $argString = "$args"
       if ($argString -match "-la\b" -or $argString -match "-al\b" -or $argString -match "-a\b") {
           Get-ChildItem -Force | Format-Table
       } elseif ($argString -match "-l\b") {
           Get-ChildItem | Format-Table
       } elseif ($args.Count -gt 0) {
           Get-ChildItem $args
       } else {
           Get-ChildItem
       }
   }

   # 2. 유용한 단축 명령어 및 함수
   function lla { ls -al }
   function ll  { ls -l }
   function la  { ls -a }
   function ..  { cd .. }
   function clear { Clear-Host }
   function touch ($file) { New-Item -ItemType File -Name $file -Force > $null }
   function grep ($pattern) { Select-String -Pattern $pattern }
   
   # 3. rm 함수 (-rf 대응)
   function rm {
       $argString = "$args"
       if ($argString -match "-rf\b" -or $argString -match "-fr\b") {
           $Target = $args | Where-Object { $_ -notmatch "^-(rf|fr)$" }
           if ($Target) { Remove-Item -Path $Target -Recurse -Force }
       } else {
           Remove-Item $args
       }
   }
   ```
3. **PowerShell 창을 완전히 껐다가 다시 켭니다.**

---

## 3. `--use-on-cd` 옵션 및 `.node-version` 활용
- 자동 버전 전환 기능 : <br/>
설정한 --use-on-cd 옵션 덕분에, 프로젝트 폴더에 .node-version 파일(내용물 예: 22)만 만들어 두면 해당 폴더로 진입 시 알아서 fnm v22 환경과 그에 맞는 격리된 글로벌 모듈들이 자동으로 매핑됩니다.

--- 

## 4. 저장 위치(`FNM_DIR`) 관리 참고 사항
- 기본값 (별도 설정 불필요):
    - `fnm`은 이 설정을 별도로 넣지 않아도 Windows 환경에 맞는 기본 저장 경로를 알아서 찾아갑니다.
    - winget으로 설치한 경우, 기본적으로 %AppData%\Roaming\fnm 경로를 Node.js 버전 저장 공간으로 사용하므로 아무런 설정을 안 해도 정상 작동합니다.
    - 설치된 Node.js의 저장 경로를 확인하고 싶다면 PowerShell에 fnm env를 입력해 출력되는 내용 중 FNM_DIR 항목에서 자동으로 잡힌 기본 경로를 확인할 수 있습니다.
    - 저장 위치를 바꾸고 싶을 때 (예: C드라이브 용량 부족):
        - Node.js 버전을 많이 설치하면 용량을 차지하므로, 기본 경로(AppData) 대신 사용자 홈 폴더(.fnm)나 D드라이브 같은 커스텀 경로에 Node.js를 강제로 몰아서 저장하고 싶을 때 FNM_DIR 환경 변수를 지정하여 활용할 수 있습니다.


--- 

## 5. 주요 명령어 요약

* **버전 설치:**
  * 최신 LTS: `fnm install --lts`
  * 특정 버전: `fnm install 22`
* **버전 확인 및 관리:**
  * 설치된 목록: `fnm list`
  * 원격 설치 가능한 목록: `fnm list-remote`
  * 현재 활성환화된 버전: `fnm current`
  * 버전 변경: `fnm use 22` 또는 `fnm use lts-latest`
  * 기본값 설정(새 터미널 열 때 기본값 지정): `fnm default 22` 또는 `fnm default system`
  * 기존 시스템 Node 전환: `fnm use system`
* **실행 경로 확인:** <br/>
  ```powershell
  where.exe node
  ```
  
  - `fnm_multishells` 경로가 나오면 fnm 환경이 정상 작동 중인 상태
  - `C:\Program Files\nodejs` 경로가 출력되면 기존 윈도우 시스템 Node 환경
  - 기존 C:\Program Files\nodejs\node.exe 경로가 나란히 있어도, 터미널은 맨 위에 있는 fnm 환경을 최우선으로 인식합니다.
  - fnm 환경에서 npm install -g 【모듈명】을 실행하면 기존 윈도우 환경과 완전히 격리된 공간에 모듈이 안전하게 설치됩니다.
    
---

## 6. VS Code 주의사항
- VS Code의 기본 터미널은 반드시 **PowerShell**을 사용해야 합니다. <br/>
  (Terminal: Select Default Profile CMD 설정 시 `fnm` 연동 과정에서 프리징 현상 발생 가능)

