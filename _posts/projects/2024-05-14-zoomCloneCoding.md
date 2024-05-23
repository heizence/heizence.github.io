---
title: Zoom Android 앱 클론코딩
description: ""
date: 2024-05-14 19:23:00 +09:00
categories: [SW Development / CS, Projects]
tags: [Java, Android]
pin: true
math: true
mermaid: true
---

오랫동안 React-Native 로만 앱 개발을 하다가 Android Native 앱 개발을 시도해 보았다. 마침 영상 스트리밍 기술도 같이 공부하고 있어서 그 부분도 적용해 볼까 하는 생각이 들었고, 간단한 화상회의 어플을 만들기로 결정했다.

## **프로젝트 소개**

화상회의 서비스 Zoom 의 기능을 비슷하게 만들어 보는 클론 코딩 프로젝트이다. 모든 기능을 다 구현하지는 않고 핵심적인 기능 위주로 구현하였다. 서비스 이름은 MeetAnywhere 로 정했다.

실제 Zoom 서비스는 회의실에 수십 명이 입장하여 미디어 정보를 실시간으로 주고받을 수 있으나, 본 프로젝트에서는 회의당 4명 이하의 소규모 인원의 미디어 정보를 처리할 수 있을 정도로만 설계하였다.

<br/>
**사용한 기술/플랫폼**
- Java, Android
- Node.js, Express, Socket.io, Mysql
- WebRTC
- AWS EC2

<br/>
실시간 스트리밍을 위해 WebRTC 기술을 사용하였다. WebRTC 를 이용한 설계 방식에는 Mesh, SFU, MCU 등 여러 가지가 있고 서비스 규모와 목적에 따라 적절한 방식을 선택할 수 있다. 본 프로젝트에서는 위에서 언급한 대로 소규모로 설계하는 것을 목표로 하였기에 가장 기본적인 Mesh 방식을 이용하여 스트리밍 기능을 구현하였다.

서버의 경우 일반적인 http 요청을 처리하는 메인 서버, WebRTC 연결 시 필요한 signaling 서버, 회의 진행 시 실시간 요청(채팅, 회의 참석, 종료 등)을 처리하는 소켓 서버를 모두 포함한다.

## **주요 기능**

#### **1.회원가입, 로그인**

서비스 사용을 위한 계정을 생성하고 로그인한다. 회원가입 시 로그인 할 때 사용할 이메일과 비밀번호, 회의 참석 시 표시할 이름을 입력해야 한다.

<br/>
<figure style="width:600px;">
<iframe src="https://drive.google.com/file/d/154NXQ3YXNO9UXOHainsXnvy_w8gEda0c/preview" width="600" height="400" allow="autoplay" allowFullScreen></iframe>
<figcaption style="text-align:center; font-size:15px;">영상1)회원가입, 로그인</figcaption>
</figure>

#### **2.회원정보 변경**

마이페이지 탭에서 회원가입 시 입력한 각종 정보들(사진, 이름, 비밀번호)을 변경할 수 있다.

<br/>
<figure style="width:600px;">
<iframe src="https://drive.google.com/file/d/1xfWNpF82RurYPObZvZb43JzFKJjkWEfJ/preview" width="600" height="400" allow="autoplay" allowFullScreen></iframe>
<figcaption style="text-align:center; font-size:15px;">영상2)프로필 사진, 이름, 비밀번호 변경</figcaption>
</figure>

#### **3.비밀번호 재발급**

비밀번호 분실 시 새로운 임시 비밀번호를 재발급 받을 수 있다. 임시 비밀번호는 가입 시 사용한 이메일로 발송된다.

#### **4.회원탈퇴**

더 이상 서비스 사용이 필요하지 않을 때 계정을 삭제하는 기능이다.

<figure style="width:600px;">
<iframe src="https://drive.google.com/file/d/1uyeeWQbGij3wgIts-hYdjYjDS1i6pCnv/preview" width="600" height="400" allow="autoplay" allowFullScreen></iframe>
<figcaption style="text-align:center; font-size:15px;">영상3)비밀번호 재발급, 회원탈퇴</figcaption>
</figure>

#### **5.회의 생성 및 참석, 회의 진행**

회의 주최자가 새 회의실을 생성하면 회의 고유 ID 가 생성되고 참가자들은 회의 ID 를 입력해서 회의에 참석할 수 있다. 단, 회의 생성은 로그인한 사용자만 가능하다. 회의 참석은 로그인 없이도 회의 ID 만 입력하면 가능하다.

참가자들은 각자의 비디오 및 마이크 스트림을 전송하고 수신한다. 회의 정보 조회, 참가자 조회, 비디오 및 마이크 on/off 기능을 사용할 수 있다.

<figure style="width:600px;">
<iframe src="https://drive.google.com/file/d/1uyzKcID7B7y2mIBSWUNpDZh3R1tQFvgP/preview" width="600" height="400" allow="autoplay" allowFullScreen></iframe>
<figcaption style="text-align:center; font-size:15px;">영상4)회의 생성 및 참석, 각종 기능 사용</figcaption>
</figure>

#### **6.실시간 채팅**

채팅 기능을 통해 회의 중 실시간으로 메시지를 주고받을 수 있다.

<figure style="width:600px;">
<iframe src="https://drive.google.com/file/d/1aoRgFNGeDuHgny0KlQU4A5gGTWHWrCTT/preview" width="600" height="400" allow="autoplay" allowFullScreen></iframe>
<figcaption style="text-align:center; font-size:15px;">영상5)실시간 채팅</figcaption>
</figure>

#### **7.참가자 퇴장, 호스트 재할당, 회의 종료**

회의에 참석중인 참가자들은 회의에서 퇴장할 수 있다.

호스트의 경우 단순 퇴장 시, 새로운 호스트를 할당해서 회의의 통제권을 다른 참가자에게 넘겨준 후에 퇴장한다. 만약 회의를 끝내고자 한다면 회의를 종료하고 회의실 데이터를 제거할 수 있다. 이 경우 다른 참가자들은 강제로 회의 퇴장 처리된다.

<figure style="width:600px;">
<iframe src="https://drive.google.com/file/d/1B47Kl8TqFLS7cX7bCyjNXi19m4tRjHrb/preview" width="600" height="400" allow="autoplay" allowFullScreen></iframe>
<figcaption style="text-align:center; font-size:15px;">영상6)참가자 퇴장, 호스트 재할당, 회의 종료</figcaption>
</figure>

#### **8.화면/화이트보드 공유**

참가자(주최자 포함)가 본인의 기기에 표시되는 화면을 다른 참가자들에게 공유한다. 화이트보드 공유 기능이 있어서 공유자의 그림, 필기가 다른 사용자들에게 공유되도록 할 수 있다.

<figure style="width:600px;">
<iframe src="https://drive.google.com/file/d/1-4coiFHmbPKWhLl5a-20-ox4IQb9Gtgm/preview" width="600" height="400" allow="autoplay" allowFullScreen></iframe>
<figcaption style="text-align:center; font-size:15px;">영상7)화면/화이트보드 공유</figcaption>
</figure>

## **후기**

그나마 제대로 된(?) Android 앱을 만들고 나니 우선은 보람이 컸다. WebRTC 기술을 학습하고 적용해 본 것도 충분히 의미있는 경험이었던 것 같다. 다만 야직 Java 나 Android OS 에 익숙하지 않아서 코드가 부실한 게 아쉽긴 하다. 초반에 설계를 했음에도 불구하고 기능이 예상보다 더 복잡해서 코드가 난잡해진 것도 있었다. 다음에는 설계에 시간을 좀 더 사용해야겠다.

WebRTC 를 사용하면서 대규모 사용자 접속을 처리할 수 있는 방식을 고민했었는데 너무 과도하게 욕심내는 것 같아서 소규모 사용자 접속만 처리할 수 있도록 기본적인 Mesh 방식을 채택하여 개발했다. 다음 프로젝트에서는 대규모 사용자 접속을 처리할 수 있는 SFU, MCU 방식을 사용하는 것에 도전해봐야겠다.

## **기타**

#### Github repositories

- [MeetAnywhere-Client](https://github.com/heizence/MeetAnywhere-Client)
- [MeetAnywhere-Server](https://github.com/heizence/MeetAnywhere-Server)
