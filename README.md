# Capture Board Selector

[블로그-OBS 없이 캡처보드+닌텐도 스위치 디스코드 소리 포함 화면 공유하기](https://pcloud.tistory.com/75)  

캡처보드의 영상과 소리를 프로세스로 재생하는 프로그램.


```sh
@echo off
chcp 65001 >nul
setlocal enabledelayedexpansion

title Capture Board Selector
echo ===============================================
echo   Capture Board Selector
echo ===============================================

rem =================================================
rem 설정 영역 — 실행 전에 직접 수정하세요
rem =================================================
set "VIDEO=StreamLine Mini+ GC311G2"
set "AUDIO=HDMI(StreamLine Mini+ GC311G2)"
rem =================================================

rem ▷ ffplay.exe 경로 확인
where ffplay >nul 2>nul
if errorlevel 1 (
  echo [ERROR] ffplay.exe를 찾을 수 없습니다.
  echo ffplay.exe가 PATH에 등록되어 있지 않거나, ffmpeg\bin 폴더 안에 없습니다.
  echo -----------------------------------------------
  echo 현재 경로: %CD%
  echo -----------------------------------------------
  pause
  exit /b
)

echo.
echo [INFO] 현재 설정된 장치:
echo [INFO] VIDEO = %VIDEO%
echo [INFO] AUDIO = %AUDIO%
echo.
echo ===============================================
rem ▷ 장치 목록 출력

echo.
echo [INFO] 장치 목록을 불러오는 중입니다...
ffmpeg -hide_banner -list_devices true -f dshow -i dummy > "%temp%\devicelist.txt" 2>&1

echo.
echo ===============================================
echo 비디오 장치 목록
echo ===============================================

for /f "tokens=2 delims=]" %%A in ('findstr /I " (video)" "%temp%\devicelist.txt"') do (
  set "line=%%A"
  set "line=!line:(video)=!"
  set "line=!line:"=!"
  set "line=!line: =!"
  echo !line!
)

echo.
echo ===============================================
echo 오디오 장치 목록
echo ===============================================

for /f "tokens=2 delims=]" %%A in ('findstr /I " (audio)" "%temp%\devicelist.txt"') do (
  set "line=%%A"
  set "line=!line:(audio)=!"
  set "line=!line:"=!"
  set "line=!line: =!"
  echo !line!
)

rem ▷ 장치 존재 여부 확인
findstr /C:"%VIDEO%" "%temp%\devicelist.txt" >nul
if errorlevel 1 (
  echo [ERROR] 비디오 장치 "%VIDEO%" 를 찾을 수 없습니다.
  echo 장치 이름을 위 목록에서 확인 후 다시 설정하세요.
  echo -----------------------------------------------
  pause
  exit /b
)

findstr /C:"%AUDIO%" "%temp%\devicelist.txt" >nul
if errorlevel 1 (
  echo [ERROR] 오디오 장치 "%AUDIO%" 를 찾을 수 없습니다.
  echo 장치 이름을 위 목록에서 확인 후 다시 설정하세요.
  echo -----------------------------------------------
  pause
  exit /b
)

echo [INFO] 장치 확인 완료!
echo -----------------------------------------------

rem ▷ FFplay 실행
ffplay -hide_banner -loglevel error -f dshow ^
 -rtbufsize 512M ^
 -use_wallclock_as_timestamps 1 ^
 -audio_buffer_size 50 ^
 -async 1 ^
 -i video="%VIDEO%":audio="%AUDIO%" ^
 -video_size 1920x1080 ^
 -vf scale=1920:1080,setdar=16/9 ^
 -window_title "Preview" ^
 -x 1920 -y 1080 ^
 -sync video ^
 -af "adelay=0|0"

echo -----------------------------------------------
echo [INFO] 프로그램이 종료되었습니다.
pause
endlocal
```
