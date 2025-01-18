이진 공간 분할법(Binary Space Partitioning, BSP)은 재귀적으로 유클리드 공간을 초평면 상의 볼록 집합으로 분할하는 기법이다.
[wikipedia - 이진 공간 분할법](https://ko.wikipedia.org/wiki/%EC%9D%B4%EC%A7%84_%EA%B3%B5%EA%B0%84_%EB%B6%84%ED%95%A0%EB%B2%95)

분할 과정으로 BSP 트리라 불리는 트리 구조가 만들어진다.
![[Pasted image 20241025120017.png]]
bsp알고리즘은 둠에서 사용되면서 유명해진 알고리즘이라고 한다.

doom의 경우 3차원처럼 보이지만 실제로는 2차원이고 그 2차원을 나타내기 위해 이 bsp알고리즘이 활용되었다고 한다.

## BSP 알고리즘을 통한 로그라이크류 방만들기
![[Pasted image 20241025120137.png]]
로그라이크 게임을 하다보면 일정한 맵을 돌려쓰는 게임도 있긴하지만 매번 새로운 맵을 생성하는 게임들이 존재한다.
매번 새로 맵을 생성할때 BSP알고리즘을 응용하여 새로 맵을 만들수 있다.

위에서 설명한 BSP알고리즘에서 둔각을 찾아 반복하여 나누는 방법이 아니라, 2차원상에서 직사각형 모양으로만 방을 나눠 매번 새로운 맵을 생성하는 방법이다.

거창하게 BSP알고리즘을 활용하여 만든다 말했지만 분할/정복을 하고 난 다음 방을 이어붙이기만 하면 되는 간단한 알고리즘이다.

## 과정

### 1. 방을 이분할한다.
- 다음의 경우 방 분할을 가로/세로 선택하는 것, 너무 한쪽으로만 분할되지않게 하는 것
- 분할비율은 4:6~6:4로 할 지에 대해서만 랜덤으로 설정했다.
```cpp
if(rLen/cLen>1 ||(cLen/rLen<=1 && rand()%2)) { //가로세로 랜덤 분할및 한쪽으로만 분할 막기
	int divideNum = (r2-r1)*(rand() % 3 + 4)/10; // 랜덤비율설정 ... 
}
```

### 2. 분할이 끝나면 방을 만든다.
- 분할된 공간에 1을 넣어 방으로 표시해준다.
- 나중에 길을 넣기 위해서 전체 직사각형 면들을 각각 -2씩 빼주어 방을 만들어준다.
```cpp
if (depth ==0 || (r2 - r1 <= 10 || c2 - c1 <= 10)) {
	for (int i = r1+2; i < r2-2; i++) {
		for (int j = c1 + 2; j < c2 - 2; j++) {
			Dungeon[i][j] = 1;
		}
	}
	return { r1+2, c1+2, r2-3, c2-3, r1+2, c1+2, r2-3, c2-3 };
}
```

### 3. 분할된 방 사이에 길을 만들어 합친다.
- 합치는 방법은 다양하겠지만 다음의 경우는 재귀를 통해 길을 좀 더 수월하게 만들기 위해서 왼쪽 분할되는 방을 1, 2, 오른쪽 분할되는 방을 3, 4라고 했을 때 2, 3을 연결시켜주었다.
![[Pasted image 20241025120754.png]]
```cpp
...
Dungeon[temp1.r4 + 1][(temp1.c3 + temp1.c4) / 2] = 4;
Dungeon[temp1.r4 + 2][(temp1.c3 + temp1.c4) / 2] = 4;
Dungeon[temp2.r1 - 1][(temp2.c1 + temp2.c2) / 2] = 4;
Dungeon[temp2.r1 - 2][(temp2.c1 + temp2.c2) / 2] = 4;
int rmin = min((temp1.c3 + temp1.c4) / 2, (temp2.c1 + temp2.c2) / 2);
int rmax = max((temp1.c3 + temp1.c4) / 2, (temp2.c1 + temp2.c2) / 2);
for (int i = rmin; i <= rmax; i++) {
	Dungeon[temp2.r1 - 2][i] = 4;
}
...
```
방을 좀더 다이나믹하게 만들기위해 디테일함을 다르게 줄 수 있겠지만
기본적인 알고리즘을 설명하면 위처럼 3단계로 간단하게 나눠 설계가 가능하다.

## 코드
```cpp
#include<stdio.h>
#include<iostream>
#include<algorithm>
#include<string>
#include<string.h>
#include<math.h>
using namespace std;

#define DungeonSize 60
int Dungeon[DungeonSize][DungeonSize];

//1.분할한다.
//﻿2. 분할이 끝나면 방을 만든다.
//3. 방을 연결한다.
typedef struct DungeonLocation {
	int r1, c1, r2, c2;
	int r3, c3, r4, c4;
};

DungeonLocation divideDungeon(int depth, int r1, int c1, int r2, int c2) {
	DungeonLocation Location; //2. 방을 만든다.
	if (depth ==0 || (r2 - r1 <= 10 || c2 - c1 <= 10)) {
		for (int i = r1+2; i < r2-2; i++) {
			for (int j = c1 + 2; j < c2 - 2; j++) {
				Dungeon[i][j] = 1;
			}
		}
		
		return { r1+2, c1+2, r2-3, c2-3, r1+2, c1+2, r2-3, c2-3 };
	}
	
	//1. 분할한다
	//3. 방을 합친다.
	int rLen = r2 - r1;
	int cLen = c2 - c1;
	DungeonLocation temp1, temp2;
	
	if (rLen / cLen > 1 || (cLen / rLen <= 1 && rand() % 2)) { // 세로분할
		int divideNum = (r2 - r1) * (rand() % 3 + 4) / 10;
		//방 분할
		temp1 = divideDungeon(depth - 1, r1, c1, r1+divideNum, c2);
		temp2 = divideDungeon(depth - 1, r1 + divideNum, c1, r2, c2);
		
		//방합치기.
		Dungeon[temp1.r4 + 1][(temp1.c3 + temp1.c4) / 2] = 4;
		Dungeon[temp1.r4 + 2][(temp1.c3 + temp1.c4) / 2] = 4;
		Dungeon[temp2.r1 - 1][(temp2.c1 + temp2.c2) / 2] = 4;
		Dungeon[temp2.r1 - 2][(temp2.c1 + temp2.c2) / 2] = 4;
		
		int rmin = min((temp1.c3 + temp1.c4) / 2, (temp2.c1 + temp2.c2) / 2);
		int rmax = max((temp1.c3 + temp1.c4) / 2, (temp2.c1 + temp2.c2) / 2);
		for (int i = rmin; i <= rmax; i++) {
			Dungeon[temp2.r1 - 2][i] = 4;
		}
	} else { // 가로분할
		int divideNum = (c2 - c1) * (rand() % 3 + 4) / 10;
		//방분할
		temp1 = divideDungeon(depth - 1, r1, c1, r2, c1 + divideNum);
		temp2 = divideDungeon(depth - 1, r1, c1 + +divideNum, r2, c2);
		
		//방합치기
		Dungeon[(temp1.r3 + temp1.r4) / 2][temp1.c4 + 1] = 3;
		Dungeon[(temp1.r3 + temp1.r4) / 2][temp1.c4 + 2] = 3;
		Dungeon[(temp2.r1 + temp2.r2) / 2][temp2.c1 - 1] = 3;
		Dungeon[(temp2.r1 + temp2.r2) / 2][temp2.c1 - 2] = 3;
		
		int rmin = min((temp1.r3 + temp1.r4) / 2, (temp2.r1 + temp2.r2) / 2);
		int rmax = max((temp1.r3 + temp1.r4) / 2, (temp2.r1 + temp2.r2) / 2);
		for (int i = rmin; i <= rmax; i++) {
			Dungeon[i][temp2.c1-2] = 3;
		}
	}
	Location.r1 = temp1.r1, Location.r2 = temp1.r2, Location.c1 = temp1.c1, Location.c2 = temp1.c2;
	Location.r3 = temp2.r3, Location.r4 = temp2.r4, Location.c3 = temp2.c3, Location.c4 = temp2.c4;
	
	return Location;
}

void createDungeon() {
	memset(Dungeon, 0, sizeof(Dungeon));
	divideDungeon(5, 0, 0, DungeonSize, DungeonSize);
}

void printDungeon() {
	for (int i = 0; i < DungeonSize; i++) {
		for (int j = 0; j < DungeonSize; j++) {
			printf("%d", Dungeon[i][j]);
		}
		printf("\n");
	}
}

int main(void) {
	createDungeon();
	printDungeon();
}
```