---
layout: post
title: "Build your CAS - part 1"
date:   2017-03-03
categories: ruby
---

Recently, our application begins to scale up, and we have many separate
services which is developed to satisfy different need of the customers. However,
since we introduce all of them as a bundle, we need to somehow
integrate all these separate services into single userbase without
couple the code together. The first step into that would be a
single-sign-on service to centralize the authentication step. This is
the first time I build this kind of SOA, or to be more trendy,
microservices architecture, so I have a lot to learn. This is my
personal note on the development of this service.

## What is CAS?

CAS stands for Central Authentication Service, a single-sign-on protocol for
the web. Its purpose is to permit a user to access multiple applications while
providing their credentials (such as userid and password) only once.
It also allows web applications to authenticate users without gaining access
to a user's security credentials, such as a password. [1]

## Why CAS?

Beside CAS, there are other solutions, such as OpenID. However, CAS has some
advantages which help you in some specific situations:

- It's simpler than most of other solutions (again, openID) when building it yourself
- It's suitable if you already have a userbase and want to build other services
around that
- You don't intend to make your service the "identity" checker for your user

## Implementation

I follow a part of Apereo
[protocol](https://apereo.github.io/cas/4.2.x/installation/Service-Management.html),
but I have changed some parts of that protocol.

### Rough idea

Sign in: Has 3 flows

  + Success

  ![Success login diagram](https://lh3.googleusercontent.com/Y5L0qhhZaJCJV5ECzDAR9LOOdy1qDxxQZGFMEPIWy7sv_HPMEErHZI5i7pgQ-w5w8lONGgm_a2fZC5zWde01pDu9Mfj7XWju8pvWl92er2jaoJg5gK-Z7mF8EXicnYOEwBCUmYVgj0rukqCnt5OsyM0f67FXbu7P9JbqL66iyeuIRV9haOJZsV84RKQJELtPeeAejuxax4bPmkEhhc8D__fm6dtLzDDdd4B1X47D4LFwoLYtuBMcNlTeEa7BI_L9FDGLMvA26iyEyZa1pnw0pcHSMcb_c4J4EXb4r8M9hqddYDzyLMrszcWP1cC_2R6f6Fg5e5pYlQhjZaRy02IKQjnmMkSw_4g18ifLQ1c5gzLsO_IT0qCMRy2FY6USqq9QuSg8qmhSZIoaScAnuFXwesOF_ztuk8bFy_6k-mYjhRmFngFzCPJErlN4JthhGAIP1sZD9_D3VKjnhagBLYla_OAGRDaZ8-GYloqRDwdmkVaabi0HzXEzikgb4Re4Zmv6l5c8D2QK-CFXyAm8i1A9t4A3arW00u9VFhiDvLVpn_ZePqXp7s6I5n5ckGa03My8MCmIBVqC39Bkjnh21XkFjaMp4BHO6KiK6wXiLfJXg4HObNfyjPxY2zlDvQTp4FxH6Kx7PLxaQnVrXJkGs38BkBDKpbtQKjQjEXT7Mnt1zw=w533-h631-no)

  + Failed when credentials are incorrect

  ![Failed login diagram - incorrect
  credential](https://lh3.googleusercontent.com/muZHESib3yY7owcmcgLOiZkz-Mq-pvcXvpN4-OZKVv6p5NQIHWHoHYJJyKcrPqIp3G6pI7a-2UmSCbrdi90sY_06SchCaN8xHszYzrLw3eUu0Y5WkjEGl6Rp76nYMa-rUUEoINsdKz3wXJUltU5q31bK0Okcef7wrHiIC2q8RHyAB2-MFdNHmtFDm534vnFOHv2YQlHWVudXHixrNW8CllztK6AGXt-Cy2kumtxIPCBQdau53ErOlZaA8khhMXTNxG07QoE0yjzBShLafcyHnyz3eQ4E89MIWZdiBHGNKyheH_3rWvZ9-7H5kc7WWUcUAK_Pbv0umWSCnjJ6vd071kpFmm1qM-dUGp15enx74L8qUGU8cHu3_eqoxwtr9G2zEJuEx2iblcg4DdiyZwb6vFRV66YW0mF1XhNYjD-F7MX6OzXi-X_nh4j7nG3C-feuL89qX5q-WdXEbfo0VeQVYtvjY5Ea9cAuQy1PRu0cU_k9uLnqKqpMLs37Usu1aZBnP9cfXDU6_CV4lKD3xJiDuiZ514jDqyTRj6LQblaKefqQtnP8pcIUZYkEr2drAjcmIEnXG6JiDy1Xal4TnZiZ45nrW8SxkeCUwPMksgVVZXIPVDpcplxF=w513-h361-no)

  + Failed when user try to reuse the validation_ticket

  ![Failed login diagram - incorrect ticket](https://lh3.googleusercontent.com/dXL9bZirGyZpHmEadY-P_9Z2FHKn1fLi008eUMsbUMc6cjF4d0t2yyEAJISlNTObxQJUK_WhBb44iGb3wkhIuQsxRtcByrn4ljzrLA92wKXS8Y0UHLSUivwD6S0DVLqcVfDd2iFfE8umY6q7FTvc2Jsi2OLeRPGV1J1rc7VFpEbqY3tXLxsgLiMJw4mZa_0w3-arSX-ButG-OHiDNBenUkVraAnxVbUhBF56BXTifv1GQoaYXNGe-RqLZSPiF7PTB11GyiO1kJjb0T53BsLzOMU1g3vhqZioyzptVn7pEiJTBfslN8NRdeYlwpuNk8C10gwjs73xEmsTCOo_OB2dXXWKTLujfQQaSI_VR2DO2Ncqpww29M5tuYOLWEBxTIZkEbNZiKRW7dnhvqKlnkYdmpOhlfspgS_oZ93Bi91X7M6xttP82hMvUgGeKru4rN6oKjAVGxGVero8dPQ42FKW74E_t_5lJA9nl9ptxm_Uy7D8hkJfBpCl-iTSohb1nl0M_bVrTLTyWsTtZoVM_TftJEl7T4kppieMSpOQsaUiM-VlBchKs44UWbeB7gsXNkRm5ER27cW4Ymax9HG-ZJhgABChdjehcATMI9BblGMw16Cl4xP5Ohjw=w533-h631-no)

Log out:

  + the protocol suggest 2 ways to implement single log out:

    1. With in-memory ticket registry and cleaner: when we issue log out, we set the long live token to
    expired, then cleaner will issue a logout callback to the authentication
    service and all apps

    2. With distributed ticket registry and no cleaner: when we issue log out,
    we'll redirect to authentication service logout url. In authentication
    service, the long-live-token will be exipired.

  + We choose the distributed ticket registry with no cleaner. However, we
  currently haven't use distributed ticket registry yet (we'll consider it after
  a while). The ticket expiration checker currently goes through HTTP
  request.

  + Implementation require 2 scenarios

    1. Logout from authentication server: clear session, then expire the
    long-live token. When app server try to check expiration of long live token,
    they will see that it's expire, lead to app server automatically clears session

      ![Log out from authentication server](https://lh3.googleusercontent.com/eVfhV_6FQ8KfBle2bYQhNFOlvkRmDPOEDmWYSLCM3jtZZACmqOIIL6GHiOvnKg18m86pJlLU404CDOWArdMLoQoEOnoEKXV1elGwsnba-LAYjKoH9qWwr0k5sMyxpkNUTNbKs5eob1i8s7JDqgmYdJ6flySZEgUJRabUD08G9MRnsgnTL0NOM79JAbZXd1fMOIQpkV1GnFJK7s7QnSGB0c6g0s3-DB14TVmtkXt6gErPEHzCh4jRLez_E01lZvus-g8-K69EWd1vqZaYeCWSsYTSOtNY64M6f6GLe0S0oQdvw5ksCF68TlNMcS6d5W4fLoK3k7IN6fOb-fv3NAPZOQt-CfhVf0DYRwRRzZxRRhS__Jm_XUbIKV98JDcDHE_fLyM_YP6znjSXhcpCntTbt5v3TWlRlOBfX6peFmKn15_MA1LnRYGECX2doq-3a_Y5nyDP0u2b0t9z4K3qwm-9-RZ5weyUUnXQcd8vJZKG8xj68CJbbRh6BlNri_6h-hydQ3qOXBmFQCKqDMTWloDhrMq0ce2s5pEcmuba8qvRP-cOm8t7MUg2_Eih2h0GRnYw47lJe1K-K6oAcxk79Dzxad6qLtuFFVY1ageomwRt9TA-f7_qzi_l=w466-h464-no)

    2. Logout from app server: clear session, redirect to log out
    from authentication server

      ![Log out from app server](https://lh3.googleusercontent.com/7RBiakMVDk-GxilhNsbMG6bSjNEJNHDi5uY5BzAbw7RVNP9MMOUFnQ53F3ZaALyn1gU3yMIYTUdxei7NZUaVyEtdGO-AWFvtepMrECMt5rW1kqHouFrtXLMIvXXbeMbjgmfBon5eZFhi2Nk22lLhhWe_fnHwRiw9C-GfebA1R6s4EP5E4pzfMUOnNMLbVyvecQJ_CxNC0u7Dlc80S-J4tKrJpvT5HJs_ZLfRgEFpAkqreYJa378qPwR5ApqDs6YDU3d96rXaxS0NSSrAfvTrnNr5HK0ictZRxme4HcaDIpwZRjx_v760pviTdZ9Bu0LNeOfMiWGELu9r3r1M4ug9FpfvJg_bDEuZXIHrPlXGpXvLxb9uTMPdY2vi_-EifR3jY7wrUK7DBk_t9Vir3lJ03liRHw94s2b3WhFdogB6Bv_d1DAhZ80ikQTSAMBWIgE1aOOsCgvEdl7-KtiK2kFkydeFgvOxDTtHiDwNdtk6ARVmxnfKVJ8xHtZmBat_-bovjq-ldc2L5F3t55vXP6rYS_wg_ZMMjLTXvudrPzTw7ai3ZlAY9zCBobo2uU6Y0Zz7LU93BdfjHOB-zZHfgAa4_hrU7D7TxF4r0mMS3IFzyARjnhcUvdDJ=w466-h271-no)

### Detail

- Coming soon...

## Reference

- [1] [Wikipedia: Central Authentication
service](https://en.wikipedia.org/wiki/Central_Authentication_Service)
