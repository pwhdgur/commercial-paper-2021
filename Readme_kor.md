# Commercial Paper Tutorial & Samples
- 어음 : 일반적으로 6-9개월 이내의 단기 자본을 얻기 위해 기존 회사(예: 대기업, 우량 기업)에서 발행하는 일종의 무담보 약속 어음

#### 시나리오
- MagnetoCorp와 DigiBank라는 두 조직이 Hyperledger Fabric 블록체인 네트워크로 대표되는 마켓플레이스인 'PaperNet'에서 상업 어음을 거래합니다.
- MagnetoCorp(Org2)의 직원인 Isabella가 회사를 대신하여 상업 문서를 발행.
- DigiBank(Org1)의 직원인 Balaji는 이 상업 어음을 구매하고 일정 기간 보유했다가 MagnetoCorp에서 소액의 이익이나 수익률로 상환.

#### 네트워크 생성
./network-starter.sh

- 도커 컨테이너 모니터링 스크립트를 실행
./organization/magnetocorp/configuration/cli/monitordocker.sh docker_test

#### 조직의 환경 설정
- Terminal 2 : 'MagnetoCorp' 창에서 다음 명령을 실행하여 해당 조직으로 작동하는 데 필요한 셸 환경 변수를 설정
cd fabric-samples/commercial-paper/organization/magnetocorp
. ./magnetocorp.sh

- Terminal 3 : 'DigiBank' 창에서 표시된 대로 다음 명령을 실행합니다.
cd fabric-samples/commercial-paper/organization/digibank
. ./digibank.sh

#### 채널에 스마트 계약 배포
## MagnetoCorp로 스마트 계약 설치 및 승인

- Terminal 2 : 'MagnetoCorp'
export FABRIC_CFG_PATH=$PWD/../../../config/

peer lifecycle chaincode package cp.tar.gz --lang node --path ./contract --label cp_0
peer lifecycle chaincode install cp.tar.gz
peer lifecycle chaincode queryinstalled
export PACKAGE_ID=cp_0:ddca913c004eb34f36dfb0b4c0bcc6d4afc1fa823520bb5966a3bfcf1808f40a
peer lifecycle chaincode approveformyorg --orderer localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name papercontract -v 0 --package-id $PACKAGE_ID --sequence 1 --tls --cafile $ORDERER_CA

## DigiBank로 스마트 계약 설치 및 승인
- Terminal 3 : 'DigiBank'
export FABRIC_CFG_PATH=$PWD/../../../config/

peer lifecycle chaincode package cp.tar.gz --lang node --path ./contract --label cp_0
peer lifecycle chaincode install cp.tar.gz
peer lifecycle chaincode queryinstalled
export PACKAGE_ID=cp_0:ddca913c004eb34f36dfb0b4c0bcc6d4afc1fa823520bb5966a3bfcf1808f40a
peer lifecycle chaincode approveformyorg --orderer localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name papercontract -v 0 --package-id $PACKAGE_ID --sequence 1 --tls --cafile $ORDERER_CA

## 채널에 체인코드 정의 커밋
- DigiBank 관리자는 다음 명령을 사용 하여 체인코드 정의 를 다음 과 같이 커밋합니다
peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --peerAddresses localhost:7051 --tlsRootCertFiles ${PEER0_ORG1_CA} --peerAddresses localhost:9051 --tlsRootCertFiles ${PEER0_ORG2_CA} --channelID mychannel --name papercontract -v 0 --sequence 1 --tls --cafile $ORDERER_CA --waitForEvent

#### 클라이언트 애플리케이션 구조
cd ~/commercial-paper/organization/magnetocorp/application
nmp install

cd ~/commercial-paper/organization/digibank/application
npm install

## 어음 발행 (MagnetoCorp에서 발행, Isabella)
- 지갑에 사용할 ID 추가
node addToWallet.js

- 어음 서류발행
node issue.js

Process issue transaction response.{"class":"org.papernet.commercialpaper","currentState":1,"issuer":"MagnetoCorp","paperNumber":"00001","issueDateTime":"2020-05-31","maturityDateTime":"2020-11-30","faceValue":5000000,"mspid":"Org2MSP","owner":"MagnetoCorp"}
MagnetoCorp commercial paper : 00001 successfully issued for value 5000000

## 어음 구입 (Digibank에서 구입, Balaji)
- 사용할 ID 추가
node addToWallet.js

- 어음 구입
node buy.js

Process buy transaction response.
MagnetoCorp commercial paper : 00001 successfully purchased by DigiBank

## 어음 상환 (Digibank에서 수행, 현재 소유자)
node redeem.js

Process redeem transaction response.
MagnetoCorp commercial paper : 00001 successfully redeemed with MagnetoCorp

## 쿼라 수행 (소유권 및 자산 내역 등..)
- History of Commercial Paper 
- Ownership of Commercial Papers
- Partial Key query, for Commercial papers in org.papernet.papers namespace belonging to MagnetoCorp
- Named Query: all redeemed papers in a state of 'redeemed' (currentState = 4)
- Named Query: all commercial papers with a face value > $4m

## 어음 발행 (MagnetoCorp에서 발행, Isabella)
node issue_2_3000000.js
node issue_3_6000000.js

## 어음 구입 (Digibank에서 구입, Balaji)
node buy_2.js
node buy_3.js

## 어음 상환 (Digibank에서 수행, 현재 소유자)
node redeem_2.js
node redeem_3.js

node queryapp.js 

#### Clean up
./network-clean.sh
