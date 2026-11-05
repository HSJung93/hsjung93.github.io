---
title: CalDAV 프로퍼티란
date: 2023-04-24 20:00:00 +0900
categories: [강의, 시니어]
tags: [CalDAV, 프로퍼티]
---

# CalDAV Property
- 캘린더 혹은 캘린더 컴포넌트(이벤트 등)에 대한 메타 정보
- Custom Property를 정의하여 사용할 수 있다. 

## 캘린더 Property
- 예를 들어 'calendar/hoesung/home' 에 위치한 캘린더의 프로퍼티 중 'getetag', 'displayname', 'max-resource-size' 프로퍼티를 다음과 같이 PROPFIND 요청할 수 있다.

```
PROPFIND /calendars/jdoe/home HTTP/1.1
Host: example.com
Content-Type: application/xml; charset=utf-8
Depth: 0

<?xml version="1.0" encoding="utf-8"?>
<D:propfind xmlns:D="DAV:">
  <D:prop>
    <D:getetag/>
    <D:displayname/>
    <C:max-resource-size xmlns:C="urn:ietf:params:xml:ns:caldav"/>
  </D:prop>
</D:propfind>
```
- 캘린더 리소스의 프로퍼티의 경우 xml 형식으로 응답 받는다.

### cf. PROPFIND 란
- CalDAV 규격에서 캘린더, 이벤트 등의 리소스의 프로퍼티를 응답받기 위한 method 이다.
- 클라이언트가 응답 받고 싶은 프로퍼티들을 특정하여 XML 양식의 바디로 요청한다.
  - 요청 양식: 프로퍼티이름, 프로퍼티이름 + 프로퍼티값 형식의 리스트

## 캘린더 컴포넌트(이벤트) 프로퍼티
- 예를 들어, `calendar/hoesung/home/1234567890.ics` 에 위치한 캘린더 이벤트의 `calendar-data` 프로퍼티를 다음과 같이 요청할 수 있다.

```
PROPFIND /calendars/jdoe/home/1234567890.ics HTTP/1.1
Host: example.com
Content-Type: application/xml; charset=utf-8
Depth: 0

<?xml version="1.0" encoding="utf-8"?>
<D:propfind xmlns:D="DAV:" xmlns:C="urn:ietf:params:xml:ns:caldav">
  <D:prop>
    <C:calendar-data/>
  </D:prop>
</D:propfind>
```

- 캘린더 컴포넌트 이벤트의 경우에는 iCalendar 형식으로 응답 받는다. 

```
HTTP/1.1 200 OK
Content-Type: text/calendar; charset=utf-8
Content-Length: 375

BEGIN:VCALENDAR
VERSION:2.0
PRODID:-//Example Corp.//CalDAV Server//EN
BEGIN:VEVENT
UID:1234567890@example.com
DTSTAMP:20230421T170000Z
DTSTART:20230501T090000Z
DTEND:20230501T100000Z
SUMMARY:Team meeting
DESCRIPTION:Weekly team meeting to discuss project updates.
LOCATION:Conference room 2
END:VEVENT
END:VCALENDAR
```

### cf. iCalendar 란
- 서버간 캘린더와 스케줄링 정보를 교환하기 위한 파일 포맷으로 RFC 5545(https://www.rfc-editor.org/rfc/rfc5545)에서 정의
- `.ics` 형식의 파일
- 이벤트(VEVENT), 할일(VTODO), 타임존(VTIMEZONE) 등 컴포넌트들의 set으로 구성
- 컴포넌트는 다시 요약(SUMMARY), 시작시간(DTSTART) 등 프로퍼티들의 set으로 구성