### 데코레이터 패턴

---

    객체에 동적으로 기능을 추가하는 패턴
    작업 과정
    1.기능을 사용할 인터페이스 정의
    2.인터페이스의 구현
    3.기능 변경에 대한 구현 추상클래스 선언
    4.기능 변경에 대한 구현 추상클래스를 상속하여 기능 변경에 대한 구체화

```java
public interface Officetel {
    void construction();
}

public class OneFloor implements Officetel {
    @Override
    public void construction() {
        System.out.println("바닥 공사");
    }
}

public class TwoFloor implements Officetel {
    @Override
    public void construction() {
        System.out.println("천장 공사");
    }
}

public abstract class Interior implements Officetel {
    Officetel officetel;

    public Interior(Officetel officetel) {
        this.officetel = officetel;
    }

    @Override
    public void construction() {
        officetel.construction();
    }
}

public class WallpaperInterior extends Interior {
    public WallpaperInterior(Officetel officetel) {
        super(officetel);
    }

    public void addInterior() {
        System.out.println("벽지 인테리어 추가");
    }

    @Override
    public void construction() {
        super.construction();
        addInterior();
    }
}

public class LightInterior extends Interior {
    public LightInterior(Officetel officetel) {
        super(officetel);
    }

    public void addInterior() {
        System.out.println("조명 인테리어 추가");
    }

    @Override
    public void construction() {
        super.construction();
        addInterior();
    }
}

public class OwnerMain {
    public static void main(String[] args) {
        System.out.println("오피스텔 1층 작업");
        Officetel oneFloorOfficetel = new OneFloor();
        oneFloorOfficetel.construction();
        System.out.println("오피스텔 2층 작업");
        Officetel twoFloorOfficetel = new WallpaperInterior(new TwoFloor());
        twoFloorOfficetel.construction();
        System.out.println("오피스텔 2층 작업 변경");
        Officetel changeTwoFloorOfficetel = new WallpaperInterior(new LightInterior(new TwoFloor()));
        changeTwoFloorOfficetel.construction();
    }
}

/**
 출력 결과 :
 오피스텔 1층 작업
 바닥 공사
 오피스텔 2층 작업
 천장 공사
 벽지 인테리어 추가
 오피스텔 2층 작업 변경
 천장 공사
 조명 인테리어 추가
 벽지 인테리어 추가
 */
```