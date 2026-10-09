# 로컬 AI 실행 환경 구축 및 보안 설정 검증

[포트폴리오 목록으로 돌아가기](../README.md)

## 목적

개인 PC에서 언어 모델을 실행하고 GPU 사용, 클라우드 비활성화, 접속 범위를 확인한다.

## 환경

| 항목 | 내용 |
|---|---|
| 운영체제 | Windows |
| CPU | 사용자 표현은 '983d'; 정확한 모델명 추가 확인 필요 |
| GPU | RTX 5080 |
| RAM | 32GB |
| 실행 도구 | Ollama |
| 모델 | qwen3:14b |
| 진행일 | 2026-10-09 |

Ollama 및 드라이버 버전은 아직 기록하지 않았다. 확인 후 추가한다.

## 수행 과정

1. Ollama Windows 설치 프로그램으로 설치했다.
2. 사용자 환경 변수에 아래 설정을 추가하고 Ollama를 재시작했다.

| 이름 | 값 | 목적 |
|---|---|---|
| OLLAMA_NO_CLOUD | 1 | Ollama 클라우드 모델 및 웹 검색 기능 비활성화 |
| OLLAMA_HOST | 127.0.0.1:11434 | 루프백 주소에서만 연결 수신 |

3. PowerShell에서 모델을 다운로드하고 실행했다.

```powershell
ollama run qwen3:14b
```

4. 한국어 설명 요청에 대한 응답을 확인했다.

## 검증 결과

### GPU 적재

```powershell
ollama ps
```

사용자가 제공한 실행 결과:

```text
NAME         ID              SIZE      PROCESSOR    CONTEXT    RUNNER
qwen3:14b    bdbd181c33f2    9.6 GB    100% GPU     4096       llamacpp
```

모델 전체가 GPU에 올라간 상태를 확인했다. 처리 속도는 측정하지 않았다.

### 클라우드 기능 비활성화

```powershell
Select-String -Path "$env:LOCALAPPDATA\Ollama\server.log" -Pattern "Ollama cloud disabled" | Select-Object -Last 3
```

사용자가 제공한 로그에 아래 문구가 포함되어 있었다.

```text
Ollama cloud disabled: true
```

### 접속 주소

```powershell
Get-NetTCPConnection -State Listen -LocalPort 11434 | Select-Object LocalAddress, LocalPort
```

예상한 127.0.0.1:11434 결과가 나왔다는 사용자 확인을 받았다. 원본 출력 및 캡처는 아직 문서에 첨부하지 않았다.

## 배운 점과 한계

- 로컬 실행은 계산 위치에 대한 설명이며 PC의 인터넷 연결 여부와는 별개다.
- 클라우드 기능을 끄는 설정은 업데이트·모델 다운로드를 포함한 모든 외부 통신을 차단하지 않는다.
- 루프백 제한은 다른 기기의 직접 접속을 제한한다. PC 내부의 다른 프로그램 접근까지 차단하는 인증 장치는 아니다.
- 오프라인 실행 시험 및 네트워크 통신 관찰은 아직 수행 기록이 없다.
- 대화 종료, 모델 메모리 해제, 프로그램 종료는 서로 다른 동작이다.

## 다음 작업

- [ ] CPU와 Ollama 버전 확인 및 기록
- [ ] 개인정보를 가린 검증 화면 첨부
- [ ] 네트워크 연결 해제 후 새 질문 응답 확인
- [ ] 짧은 동일 질문을 사용해 응답 시간 측정

## 공식 참고 자료

- [Ollama Windows 설치](https://docs.ollama.com/windows)
- [설정 및 GPU 확인 FAQ](https://docs.ollama.com/faq)
- [Qwen3 14B 모델](https://ollama.com/library/qwen3:14b)
