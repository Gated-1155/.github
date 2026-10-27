# Gated-1155

**기본은 잠겨 있고, 관리자가 열어야 움직이는 ERC-1155 배지 토이 프로젝트.**

배지는 발급(mint)과 소각(burn)만 자유롭고, 사용자 간 전송과 승인(operator approval)은 기본적으로 모두 차단됩니다. 관리자가 컨트랙트 전체에 걸린 `transferable` 스위치를 켜야만 전송이 가능해집니다. "어떤 함수를 막아야 정말 못 옮기는가"와 "막았다는 걸 어떻게 증명하는가"가 이 프로젝트의 중심 질문이었습니다.

## 설계 한눈에 보기

| 항목 | 내용 |
| --- | --- |
| 전송 차단 지점 | OpenZeppelin v5의 `_update` 한 곳. 직접 전송, 배치 전송, operator 경유 전송이 모두 여기로 모입니다. |
| 승인 차단 | `setApprovalForAll`은 항상 `ApprovalNotAllowed()`로 revert. 지금은 무해한 승인이 스위치를 켜는 순간 살아나는 것을 막기 위함입니다. |
| 전송 스위치 | `transferable` 단일 불리언(기본 `false`), 관리자(`DEFAULT_ADMIN_ROLE`)만 변경 가능 |
| 권한 분리 | `DEFAULT_ADMIN_ROLE` / `MINTER_ROLE` / `PAUSER_ROLE` |
| 메타데이터 | 배지 등록 시 URI 형식 검증(`https://`, `.json`). 한 번 등록한 URI는 수정 불가 |
| 재진입 | CEI 순서 덕분에 구조적으로 안전함을 공격용 컨트랙트(`MaliciousReceiver`)로 확인 |
| 백엔드 | 없음. 관리자 지갑이 프론트에서 직접 서명 |

## 저장소

| 저장소 | 역할 |
| --- | --- |
| [**onchain-contract**](https://github.com/Gated-1155/onchain-contract) | **메인.** `BadgeToken` 컨트랙트, Hardhat 3 + ethers v6 + Mocha 테스트(24개), 배포 스크립트 |
| [**onchain-client**](https://github.com/Gated-1155/onchain-client) | React + wagmi 프론트. 지갑 연결, 배지 조회, 관리자 패널(발급·권한·일시정지·전송 스위치), 트랜잭션 상태 4종(대기/확정/실패/사용자 거절) 처리 |
| [erc-1155](https://github.com/Gated-1155/erc-1155) | 레거시. 같은 컨트랙트의 초기 버전(viem + `node:test`). 배지별 `transferable` 플래그를 쓰던 시기의 구현 |
| [metadata-test](https://github.com/Gated-1155/metadata-test) | 배지 메타데이터(JSON)와 SVG 이미지 샘플 |

## 배포 정보

- 네트워크: Polygon Amoy (chain ID `80002`)
- 컨트랙트: [`0x4a69e03529aDDC8bEfB827F27cbADd6beF645Cf2`](https://amoy.polygonscan.com/address/0x4a69e03529aDDC8bEfB827F27cbADd6beF645Cf2)
- 소스 검증: [Sourcify](https://sourcify.dev/server/repo-ui/80002/0x4a69e03529aDDC8bEfB827F27cbADd6beF645Cf2)

## 기술 스택

- **컨트랙트**: Solidity, OpenZeppelin Contracts v5, Hardhat 3, ethers v6, TypeScript
- **프론트**: React 19, Vite, wagmi, viem, TanStack Query, TypeScript

## 이 프로젝트에서 신경 쓴 것

- **실패 시나리오 우선**: 기능을 늘리기보다 우회 경로를 찾는 데 시간을 썼습니다. 막힌 경로 목록과 근거, 재현 테스트를 함께 두었습니다.
- **단일 admin 키 리스크 인지**: 업그레이드 불가 컨트랙트에서 admin 키를 잃으면 복구할 수 없다는 점을 문서화했습니다.
- **재배포 시 새 주소**: 배포 스크립트는 journal이 없어 재실행하면 새 주소가 생깁니다. 이를 감수하고 README에 주소를 기록하는 방식으로 관리합니다.
