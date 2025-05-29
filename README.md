# front-s3-cf-pipeline

![img](/public/readmd/cloudfront.png)

AWS S3, AWS CloudFront, GitHub Action로 서비스 배포 파이프라인을 구성하는 테스트 레포지토리입니다.

## 목차

[주요 과정](#주요-과정)

[0. Next.js 프로젝트 구성](#0-nextjs-프로젝트-구성)

[1. AWS S3 구성](#1-aws-s3-구성)

[2. AWS CloudFront 구성](#2-aws-cloudfront-구성)

[3. AWS IAM 구성](#3-aws-iam-구성)

[4. GitHub Actions 구성](#4-github-actions-구성)

[주요 링크](#주요-링크)

## 주요 과정

### 0. Next.js 프로젝트 구성

**🔨 실습**

1. 빌드 결과로 정적 파일들이 생성될 수 있게, `next.config.ts`에서 output 옵션을 `export`로 설정합니다.

```
// next.config.js
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  /* config options here */
  output: "export",
};

export default nextConfig;
```

2. `next build`로 빌드 후, `out` 폴더에서 빌드 결과물을 확인합니다.

```
.
├── 404.html
├── _next
├── favicon.ico
├── file.svg
├── globe.svg
├── index.html
├── index.txt
├── next.svg
├── vercel.svg
└── window.svg
```

### 1. AWS S3 구성

AWS S3는 Simple Storage Service의 약자로, 확장 가능한 클라우드 스토리지 서비스를 의미합니다. 정적 파일을 호스팅하고 배포하는 데 사용됩니다. AWS S3에서 데이터를 저장하는 기본 컨테이너를 Bucket(버킷)이라고 합니다. HTML, CSS, JavaScript 파일과 같은 정적 자산을 저장할 수 있습니다. 각 버킷은 고유한 이름을 가지며, 정적 웹사이트 호스팅 기능을 활성화할 수 있습니다.

**🔨 실습**

❗️ AWS console 계정이 없다면, root 계정으로 회원가입합니다. 정보 유출로 인한 비용 문제 등을 방지하려면, [MFA](https://docs.aws.amazon.com/IAM/latest/UserGuide/enable-mfa-for-root.html)도 반드시 설정해야 합니다. ❗️

1. 버킷을 생성합니다.
2. 테스트를 위해, `out` 폴더 내의 빌드 결과물을 모두 업로드 합니다.
3. 정적 웹 사이트 호스팅을 설정합니다.
   Next.js에 맞춰서 다음과 같이 수정합니다.

- 인덱스 문서 : `index.html`
- 오류 문서 : `404.html`

4. '버킷 웹 사이트 엔드포인트'로 접속하여 확인합니다. 시간이 걸릴 수 있으니, 바로 보이지 않는다면 잠시 기다립니다.

### 2. AWS CloudFront 구성

AWS CloudFront는 Amazon의 글로벌 콘텐츠 전송 네트워크(CDN) 서비스입니다. S3에 저장된 콘텐츠를 전 세계 사용자에게 빠르고 안전하게 전달하기 위해 사용됩니다.

- CDN

CDN(Content Delivery Network)은 지리적으로 분산된 서버 네트워크를 통해 웹 콘텐츠를 사용자에게 효율적으로 전달하는 시스템입니다. 사용자의 위치와 가장 가까운 서버에서 콘텐츠를 제공함으로써 로딩 속도를 향상시키고 서버 부하를 분산시킵니다.

- 캐시 무효화

캐시 무효화(Cache Invalidation)는 저장된 기존 콘텐츠를 강제로 삭제하여 새로운 콘텐츠로 업데이트하는 과정입니다. 웹사이트를 배포할 때 기존 캐시된 파일들이 남아있으면 사용자들이 이전 버전의 콘텐츠를 보게 될 수 있습니다. 따라서 새로운 배포 후에는 캐시 무효화를 실행하여 모든 사용자가 최신 버전의 콘텐츠를 받을 수 있도록 해야 합니다. CloudFront에서는 특정 경로나 전체 콘텐츠에 대해 무효화를 수행할 수 있습니다.

**🔨 실습**

1. CloudFront에서 `배포 생성` 버튼을 눌러 이전 단계의 S3를 지정합니다.
2. '일반' 메뉴의 '배포 도메인 이름'으로 접속하여 확인합니다. 시간이 걸릴 수 있으니, 바로 보이지 않는다면 잠시 기다립니다.

### 3. AWS IAM 구성

AWS IAM(Identity and Access Management)은 AWS 리소스에 대한 접근을 안전하게 제어하는 서비스입니다. GitHub Actions에서 AWS 서비스에 접근하기 위해서는 적절한 권한을 가진 IAM 사용자나 역할을 생성해야 합니다. S3 버킷에 파일을 업로드하고, CloudFront 배포를 무효화할 수 있는 최소한의 권한만을 부여하여 보안을 강화합니다.

**🔨 실습**

1. 새 '정책'을 만듭니다. 파이프라인 구성에 필요한 CloudFront, S3 관련 권한만 허용합니다.
2. 새 '사용자'를 만듭니다.

- 'AWS Management Console에 대한 사용자 액세스 권한 제공' 을 선택합니다.
  `IAM 사용자를 생성하고 싶음`를 선택합니다.

- 다음 단계에서, `직접 정책 연결`를 선택해 새로 생성한 정책과 연결합니다.

### 4. GitHub Actions 구성

GitHub Actions는 GitHub에서 제공하는 CI/CD(Continuous Integration/Continuous Deployment) 플랫폼입니다. 코드 변경사항이 발생할 때마다 자동으로 빌드, 테스트, 배포 과정을 실행할 수 있습니다. YAML 파일로 워크플로우를 정의하며, 다양한 이벤트(push, pull request 등)에 반응하여 작업을 수행합니다.

- 레포지토리 secret과 환경변수

GitHub Repository Secrets는 민감한 정보(API 키, 패스워드, 토큰 등)를 안전하게 저장하고 GitHub Actions 워크플로우에서 사용할 수 있게 해주는 기능입니다. 이러한 값들은 암호화되어 저장되며, 워크플로우 실행 중에만 접근 가능합니다. AWS 자격 증명, S3 버킷 이름, CloudFront 배포 ID 등을 secrets로 저장하여 코드에 하드코딩하지 않고도 안전하게 사용할 수 있습니다. 환경변수로 설정하여 ${{ secrets.SECRET_NAME }} 형태로 워크플로우에서 참조할 수 있습니다.

**🔨 실습**

1. secrets 추가
   'Settings' - 'Secrets and variables' - 'Actions'에서, `New repository secret` 버튼을 클릭해 다음 secrets를 추가합니다.

```
AWS_REGION
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
S3_BUCKET_NAME
CLOUDFRONT_DISTRIBUTION_ID
```

2. deployment.yml 작성

deployment.yml 파일을 다음과 같이 생성하여 GitHub workflow를 구성합니다.

```
name: Deploy Next.js to S3 and invalidate CloudFront

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  deploy:
    # 0. Ubuntu 최신 환경을 세팅합니다.
    runs-on: ubuntu-latest

    # 1. Checkout 액션을 통해 코드를 가져옵니다.
    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    # 2. 프로젝트 의존성을 설치합니다.
    - name: Install dependencies
      run: npm ci

    # 3. Next.js 프로젝트를 빌드합니다.
    - name: Build
      run: npm run build

    # 4. AWS 자격 증명을 구성합니다.
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ secrets.AWS_REGION }}

    # 5. Next.js 빌드 결과물을 S3에 업로드합니다.
    - name: Deploy to S3
      run: |
        aws s3 sync out/ s3://${{ secrets.S3_BUCKET_NAME }} --delete

    # 6. CloudFront 캐시를 무효화합니다.
    - name: Invalidate CloudFront cache
      run: |
        aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} --paths "/*"
```

## 주요 링크

- S3 버킷 웹사이트 엔드포인트: http://hanghae-front-9.s3-website.ap-northeast-2.amazonaws.com/
- CloudFrount 배포 도메인 이름: https://d2xm35e5plqmno.cloudfront.net
