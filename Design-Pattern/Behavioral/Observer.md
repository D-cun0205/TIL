### 옵저버 패턴

---

    주체(특정 객체)의 상태변화를 공유하는 패턴

```java
public interface Computer {
    String msg();
    void addProcess(Process process);
    void notifyToProcess(String msg);
}

public class Notebook implements Computer {
    List<Process> processList = new ArrayList<>();

    @Override
    public String msg() {
        return "노트북 메시지 >>> ";
    }

    @Override
    public void addProcess(Process process) {
        this.processList.add(process);
    }

    @Override
    public void notifyToProcess(String msg) {
        this.processList.forEach(process -> process.notifyMsg(msg() + msg));
    }
}

public interface Process {
    String name();
    void notifyMsg(String msg);
}

public class Kakaotalk implements Process {
    @Override
    public String name() {
        return "카카오톡이 노트북에게 받은 변경 사항 : ";
    }

    @Override
    public void notifyMsg(String msg) {
        System.out.println(name() + msg);
    }
}

public class Chrom implements Process {
    @Override
    public String name() {
        return "크롬이 노트북에게 받은 변경 사항 : ";
    }

    @Override
    public void notifyMsg(String msg) {
        System.out.println(name() + msg);
    }
}

public class Main {
    public static void main(String[] args) {
        Kakaotalk kakaotalk = new Kakaotalk();
        Chrom chrom = new Chrom();
        Notebook notebook = new Notebook();
        notebook.addProcess(kakaotalk);
        notebook.addProcess(chrom);
        notebook.notifyToProcess("Change OS from Windows to Mac OS");
    }
}

/**
 출력 내용 :
 카카오톡이 노트북에게 받은 변경 사항 : 노트북 메시지 >>> Change OS from Windows to Mac OS
 크롬이 노트북에게 받은 변경 사항 : 노트북 메시지 >>> Change OS from Windows to Mac OS
 */
```