## Tile Palette

- Tilemap 오브젝트에 배치할 Tile Asset들을 등록해두는 저장소
- Palette에 있는 Tile Asset을 여러 속성에 따라 배치하거나 삭제할 수 있다.
- ex) 현실에서 그림을 그리기 위해 물감을 짜두는 팔레트와 동일한 기능

## Tile Asset

- 맵에 배치할 수 있는 하나의 타일 (Sprite, Color, Collider Type과 같은 속성을 설정 가능)
- ex) 팔레트에 짜는 물감

## Grid 오브젝트

- 자식으로 배치되는 Tilemap 오브젝트들의 관리자
- Cell Layout, Cell Swizzle 정보를 이용해 타일이 배치되는 Tilemap의 레이아웃 설정

## Tilemap 오브젝트

- Tile Asset을 배치하고, 실제 게임에 출력
- Grid 오브젝트의 자식으로 여러 개의 Tilemap 오브젝트 등록 가능 (Layer 구분)
- ex) 그림을 그리는 도화지