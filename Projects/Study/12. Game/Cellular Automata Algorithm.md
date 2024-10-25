격자 구조로 쪼개진 세포(단위)들이 주변에 있는 세포에 따라 자신의 상태를 변화시키는 알고리즘이다.

## 생성 과정

1. 특정한 비율(ex. 50%)로 맵을 벽으로 채운다
2. 맵의 각 타일을 선택하여 주변 8칸 중 벽이 4칸을 초과할 경우 벽으로, 4칸 미만일 경우 길로 바꾼다.
3. 2번 과정을 정해진 수(ex. 5회)만큼 반복한다.

| ![[Pasted image 20241025113837.png]] | ![[Pasted image 20241025113857.png]] | ![[Pasted image 20241025113859.png]] |
| ------------------------------------ | ------------------------------------ | ------------------------------------ |
| 1번 과정                                | 2번 과정                                | 3번 과정                                |

## 코드

```cs
using UnityEngine;
using UnityEngine.Tilemaps;

public class CaveGeneratorByCellularAutomata : MonoBehaviour
{
	[SerializeField] private int width;
	[SerializeField] private int height;
	
	[SerializeField] private string seed;
	[SerializeField] private bool useRandomSeed;
	
	[Range(0, 100)]
	[SerializeField] private int randomFillPercent;
	[SerializeField] private int smoothNum;
	
	private int[,] map;
	private const int ROAD = 0;
	private const int WALL = 1;
	
	[SerializeField] private Tilemap tilemap;
	[SerializeField] private Tile tile;
	[SerializeField] private Color[] colors;
	
	private void Update()
	{
		if (Input.GetMouseButtonDown(0))
			GenerateMap();
	}
	
	private void GenerateMap()
	{
		map = new int[width, height];
		MapRandomFill();

		//반복이 많을수록 동굴의 경계면이 매끄러워진다.
		for (int i = 0; i < smoothNum; i++)
			SmoothMap();
	}

	//맵을 비율에 따라 벽 혹은 빈 공간으로 랜덤하게 채우는 메소드
	private void MapRandomFill()
	{
		if (useRandomSeed)
			seed = Time.time.ToString(); //시드
		
		System.Random pseudoRandom = new System.Random(seed.GetHashCode()); //시드로 부터 의사 난수 생성
		
		for (int x = 0; x < width; x++)
		{
			for (int y = 0; y < height; y++)
			{
				if (x == 0 || x == width - 1 || y == 0 || y == height - 1)
					map[x, y] = WALL; //가장자리는 벽으로 채움
				else
					map[x, y] = (pseudoRandom.Next(0, 100) < randomFillPercent) ? WALL : ROAD; //비율에 따라 벽 혹은 빈 공간 생성
				
				OnDrawTile(x, y); //타일 생성
				SetTileColor(x, y); //타일 색상 설정
			}
		}
	}
	
	private void SmoothMap()
	{
		for (int x = 0; x < width; x++)
		{
			for (int y = 0; y < height; y++)
			{
				int neighbourWallTiles = GetSurroundingWallCount(x, y);
				if (neighbourWallTiles > 4)
					map[x, y] = WALL; //주변 칸 중 벽이 4칸을 초과할 경우 현재 타일을 벽으로 바꿈
				else if (neighbourWallTiles < 4)
					map[x, y] = ROAD; //주변 칸 중 벽이 4칸 미만일 경우 현재 타일을 빈 공간으로 바꿈
				
				SetTileColor(x, y); //타일 색상 변경
			}
		}
	}
	
	private int GetSurroundingWallCount(int gridX, int gridY)
	{
		int wallCount = 0;
		//현재 좌표를 기준으로 주변 8칸 검사
		for (int neighbourX = gridX - 1; neighbourX <= gridX + 1; neighbourX++)
		{
			for (int neighbourY = gridY - 1; neighbourY <= gridY + 1; neighbourY++)
			{
				//맵 범위를 초과하지 않게 조건문으로 검사
				if (neighbourX >= 0 && neighbourX < width && neighbourY >= 0 && neighbourY < height)
				{
					//벽은 1이고 빈 공간은 0이므로 벽일 경우 wallCount 증가
					if (neighbourX != gridX || neighbourY != gridY)
						wallCount += map[neighbourX, neighbourY];
				}
				else
					wallCount++; //주변 타일이 맵 범위를 벗어날 경우 wallCount 증가
			}
		}
		
		return wallCount;
	}
	
	private void SetTileColor(int x, int y)
	{
		Vector3Int pos = new Vector3Int(-width / 2 + x, -height / 2 + y, 0); //화면 중앙 정렬
		tilemap.SetTileFlags(pos, TileFlags.None); //타일 색상을 수정하기 위해 TileFlags를 None으로 설정
		
		switch (map[x, y])
		{
			case ROAD:
				tilemap.SetColor(pos, colors[0]);
				break;
			case WALL:
				tilemap.SetColor(pos, colors[1]);
				break;
		}
	}
	
	private void OnDrawTile(int x, int y)
	{
		Vector3Int pos = new Vector3Int(-width / 2 + x, -height / 2 + y, 0); 
		tilemap.SetTile(pos, tile);
	}
}
```
