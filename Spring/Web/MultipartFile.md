### MultipartFile

---

    form 전송시 enctype : multipart/form-data 설정 후 파일을 submit 하면
    서버에서 받을 때 MultipartFile 타입으로 받아서 처리할 수 있다
    html 에 ${message} 는 AddFlashAttribute 를 이용하여 Model 객체로 넘겨서
    타임리프 표현식으로 사용자에게 업로드 되었다는 메세지를 출력한다
    

파일 등록 화면

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Upload File</title>
</head>
<body>
<div th:if="${message}">
    <h2 th:text="${message}"></h2>
</div>
<form method="POST" enctype="multipart/form-data" action="#" th:action="@{/file}">
    File: <input type="file" name="file" />
    <input type="submit" value="Upload">
</form>
</body>
</html>
```

파일 등록 처리 

```java
@Controller
public class FileController {

    @GetMapping("/file")
    public String getFileView(Model model) {
        return "file/index";
    }

    @PostMapping("/file")
    public String uploadFile(
            @RequestParam MultipartFile file,
            RedirectAttributes attributes,
            Model model) {
        String fileName = file.getOriginalFilename() + " File Upload";
        attributes.addFlashAttribute("message", fileName);
        return "redirect:/file";
    }
}
```