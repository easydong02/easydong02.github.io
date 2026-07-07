---
title: "[SpringBoot] 파일 업로드/다운로드"
date: 2023-12-19 00:00:00 +0900
categories: [Backend, Spring]
tags: [spring, fileio]
render_with_liquid: false
---

## 📌 들어가며

이번 글에서는 스프링에서 **파일 업로드·다운로드**를 다룬다. 먼저 전송 방식을 이해하고, 서블릿·스프링 방식으로 업로드한 뒤, 파일 다운로드까지 구현한다.

---

## 1. 전송 방식 2가지

| 방식 | 특징 |
|------|------|
| `application/x-www-form-urlencoded` | 기본. 문자만 (`username=kim&age=20`) |
| **`multipart/form-data`** | **파일(바이너리) + 문자** 함께 전송 |

> 💡 파일은 문자가 아닌 **바이너리 데이터**라, 폼에 `enctype="multipart/form-data"`를 지정해야 한다. 이때 각 항목에 **`Content-Disposition`** 헤더가 붙고, 파일은 파일명·Content-Type과 함께 바이너리로 전송된다.

```html
<form th:action method="post" enctype="multipart/form-data">
    <li>상품명 <input type="text" name="itemName"></li>
    <li>파일 <input type="file" name="file"></li>
    <input type="submit"/>
</form>
```

---

## 2. 서블릿 파일 업로드 (Part)

```java
@PostMapping("/upload")
public String saveFile(MultipartHttpServletRequest request) throws IOException {
    String itemName = request.getParameter("itemName");   // 문자는 getParameter
    Collection<Part> parts = request.getParts();          // 파일은 Part로

    for (Part part : parts) {
        // 파일이면 저장
        if (StringUtils.hasText(part.getSubmittedFileName())) {
            String fullPath = fileDir + part.getSubmittedFileName();
            part.write(fullPath);
        }
    }
    return "upload-form";
}
```

| `Part` 메소드 | 역할 |
|------|------|
| `getSubmittedFileName()` | 클라이언트가 보낸 파일명 |
| `getInputStream()` | 전송 데이터 읽기 |
| `write(경로)` | 파일로 저장 |

> 💡 multipart 요청이면 서블릿 컨테이너가 `HttpServletRequest`를 **`StandardMultipartHttpServletRequest`**로 바꿔준다(멀티파트 리졸버). 다만 실무에서는 더 편한 **`MultipartFile`**을 쓰므로 이 방식은 잘 안 쓴다.

---

## 3. 스프링 파일 업로드 (MultipartFile)

스프링 부트는 **`MultipartFile`** 인터페이스로 훨씬 간단히 처리한다.

```java
@PostMapping("/upload")
public String saveFile(@RequestParam String itemName,
                       @RequestParam MultipartFile file) throws IOException {
    if (!file.isEmpty()) {
        String fullPath = fileDir + file.getOriginalFilename();
        file.transferTo(new File(fullPath));   // 저장
    }
    return "upload-form";
}
```

| `MultipartFile` 메소드 | 역할 |
|------|------|
| `getOriginalFilename()` | 업로드 파일명 |
| `transferTo(File)` | 파일 저장 |

> 💡 서블릿 `Part`보다 코드가 훨씬 간결하다. 실무에서는 이 방식을 쓴다.

---

## 4. 파일 다운로드

**뷰(JSP)** — 다운로드 클릭 시 파일 정보를 hidden form에 담아 전송.

```javascript
// 클릭 시 파일 정보(경로/원본명/저장명)를 form에 담아 submit
file.addEventListener("click", function() {
    document.querySelector("[name='sfolder']").value = file.getAttribute("sfolder");
    document.querySelector("[name='ofile']").value = file.getAttribute("ofile");
    document.querySelector("[name='sfile']").value = file.getAttribute("sfile");
    document.querySelector("#downform").submit();
});
```

| 파라미터 | 의미 |
|:---:|------|
| `sfolder` | 저장 경로 |
| `ofile` | 원본 파일명(화면 표시용) |
| `sfile` | 실제 저장된 (고유화된) 파일명 |

**모델(AbstractView)** — 응답 헤더를 세팅하고 파일을 스트림으로 전송.

```java
public class FileDownLoadView extends AbstractView {
    @Override
    protected void renderMergedOutputModel(Map<String, Object> model,
            HttpServletRequest request, HttpServletResponse response) throws Exception {
        // ... 파일 경로로 File 객체 생성 ...
        response.setContentLength((int) file.length());

        // IE는 인코딩을 다르게 처리
        String header = request.getHeader("User-Agent");
        boolean isIE = header.contains("MSIE") || header.contains("Trident");
        String fileName = isIE
            ? URLEncoder.encode(originalFile, "UTF-8").replaceAll("\\+", "%20")
            : new String(originalFile.getBytes("UTF-8"), "ISO-8859-1");

        // attachment → 다운로드 창, inline → 웹에서 바로 열기
        response.setHeader("Content-Disposition", "attachment; filename=\"" + fileName + "\";");
        response.setHeader("Content-Transfer-Encoding", "binary");   // 파일이므로 binary

        FileCopyUtils.copy(new FileInputStream(file), response.getOutputStream());  // from → to
    }
}
```

> 💡 핵심 헤더 두 가지: **`Content-Disposition: attachment`**면 다운로드 창이 뜨고(`inline`이면 브라우저에서 바로 열림), **`Content-Transfer-Encoding: binary`**로 바이너리 전송을 설정한다. **IE는 인코딩을 별도로 처리**해야 한다.

---

## 📝 정리

```
스프링 파일 I/O
├─ 전송      multipart/form-data (파일+문자)
├─ 서블릿    Part.getInputStream()/write()
├─ 스프링    MultipartFile.transferTo() (권장)
└─ 다운로드  Content-Disposition:attachment + FileCopyUtils.copy
```

| 개념 | 한 줄 정의 |
|------|------|
| **multipart/form-data** | 파일 전송 방식 |
| **MultipartFile** | 스프링의 파일 추상화 |
| **Content-Disposition** | 다운로드/인라인 결정 |

파일 처리의 핵심은 **업로드는 `MultipartFile`, 다운로드는 응답 헤더(Content-Disposition) 세팅**이다. 서블릿 `Part`보다 스프링 `MultipartFile`이 훨씬 간결하니, 실무에서는 이를 기본으로 쓰자.
