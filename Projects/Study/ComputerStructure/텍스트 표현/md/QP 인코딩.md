출력 가능하게 변경한 인코딩(Quoted-Printable encoding, QP encoding)은 **8비트 데이터를 7비트 데이터만 지원하는 통신 경로를 통해 송수신하기 위한 인코딩 방법**이다.
QP 인코딩은 전자우편 첨부를 처리하기 위해 만들어졌다.

이 인코딩을 사용하면 = 다음에 바이트의 각 니블을 표현하는 16진 숫자 2개를 추가해 8비트 값을 표현한다.
물론 이로 인해 = 가 특별한 의미를 지니기 때문에 QP에서 '='를 표현하려면 =3D를 사용해야 한다. 또 줄의 맨 끝에 탭과 공백 문자가 온다면, 이를 각각 =09와 =20으로 표현해야만 한다.
인코딩 된 데이터는 한 줄이 76자를 넘을 수 없다. 어떤 줄의 맨 뒤가 =로 끝나면 가짜 줄바꿈을 뜻하며, 수신 쪽에서 QP로 인코딩된 데이터를 디코딩할 때는 =를 제거하고 해석한다.