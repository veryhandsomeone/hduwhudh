#include<stdio.h>
#include<stdlib.h>
#include<Windows.h>
#include<time.h>
#include<math.h>
#include<conio.h>
#define L 24  //迷宫长度为22，迷宫外有一圈ROUTE，保证迷宫自动生成时的程序运行，一会看到
#define WALL  0
#define ROUTE 1
static int Rank = 0;

int flag = 1;

typedef struct //栈中结构体
{
    int x;
    int y;
    int di;  //表示往哪个方向走
}Box;

typedef struct //栈
{
    Box* base;
    Box* top;
}Sqstack;

void CreateMaze(int Maze[][L]); //创建迷宫
void Move(int i, int j, int di,int *r,int *l); //移动方向
int Initstack(Sqstack* S);  //栈的初始化
void Push(Sqstack* S, Box e);  //入栈
void Pop(Sqstack* S,Box *e);  //出栈
void MazePath(int Maze[][L], Box start, Box end,Sqstack *S);  //自动寻路
void Show(int Maze[][L],int i,int j);  //显示迷宫
void Auto(int Maze[][L]);  //自动寻路板块
void Show_Last(int Maze[][L]);//显示最终路径

void CreateMaze(int Maze[][L])
{
    char c;
    int i,j;
    for (i=0;i<L;i++)
    {
        for (j=0;j<L;j++)
        {
            Maze[i][j]=WALL;
        }
    }
    printf("自动生成迷宫请输入'A',手动生成迷宫请输入'B'\n");
    scanf("%c",&c);
    getchar();
    while(c!='A'&&c!='B')  //防止输入错误字符导致程序停止
    {
        printf("请您重新输入:");
        scanf("%c",&c);
        getchar();
    }
    if(c=='A')
    {
        for (i = 1; i < L - 1; i++) //随机生成迷宫
        {
            for (j = 1; j < L - 1; j++)
            {
                Maze[i][j] = rand() % 2;
            }
        }
    }
    else
    {
        printf("输入'0'表示墙，输入'1'表示路:（最外圈墙之内的迷宫）\n");
        for (i = 1; i < L - 1; i++) //手动生成迷宫
        {
            for (j = 1; j < L - 1 ; j++)
            {
                scanf("%c", &c);
                Maze[i][j] = c - '0';
            }
        }
    }
    Maze[1][1] = ROUTE;
    Maze[L-2][L-2] = 3;
    Show(Maze,L,L);
}

int Initstack(Sqstack* S) //栈的初始化
{
    S->base = (Box*)malloc(sizeof(Box) * 1000);
    if (!S->base)   return 0;
    S->top = S->base;
    return 1;
}

void Push(Sqstack* S, Box e) //入栈
{
    *S->top = e;
    S->top++;

}
void Pop(Sqstack* S,Box *e) //出栈
{
    if(S->top == S->base)
    {
        printf("栈空");
    }
    else{
        *e = *--S->top; //调用栈顶元素，传回实参
    }
}

void Move(int i, int j, int di, int* r, int* l)
{
    if (di == 0) {  //右下
        *r = i + 1;
        *l = j + 1;
    }
    if (di == 1) {  //下
        *r = i + 1;
        *l = j;
    }
    if (di == 2) {  //右
        *r = i;
        *l = j + 1;
    }
    if (di == 3) {  //右上
        *r = i - 1;
        *l = j + 1;
    }
    if (di == 4) {  //左下
        *r = i + 1;
        *l = j - 1;
    }
    if (di == 5) {  //左
        *r = i;
        *l = j - 1;
    }
    if (di == 6) {  //上
        *r = i - 1;
        *l = j;
    }
    if (di == 7) {  //左上
        *r = i - 1;
        *l = j - 1;
    }
}

void MazePath(int Maze[][L], Box start, Box end,Sqstack *S)
{
    int i, j;   //当前位置
    int r, l;   //下一个可能出现的位置
    int di;   //方向
    Box cornow;  //出入栈时用的元素
    i = 1;
    j = 1;
    di = 0;
    Maze[i][j] = 2;//标识为已走过
    cornow.x = 1;
    cornow.y = 1;
    cornow.di = 0;    //迷宫初始位置和方向
    Push(S, cornow);
    do
    {
        Pop(S,&cornow); //弹出栈顶元素
        system("cls");
        Show(Maze,cornow.x,cornow.y);
        Sleep(200);
        i = cornow.x;
        j = cornow.y;
        di = cornow.di;
    while (di < 8)
    {
        Move(i, j, di, &r, &l);
        if (Maze[r][l] == 1) {
            cornow.x = i;
            cornow.y = j;
            cornow.di = di;
            Maze[r][l] = 2;
            Push(S, cornow); //将移动前的节点信息压入栈中
            i = r;
            j = l;
            di = -1; //下一节点仍从0号方向开始试探
            system("cls");
            Show(Maze,i,j);
            Sleep(200);
            if (i == end.x && j == end.y) {
                do{
                    Pop(S,&cornow);
                    Maze[cornow.x][cornow.y] = 3;
                }while(S->top > S->base);  //将栈中元素进行标记
                system("cls");
                Show_Last(Maze);
                exit(1);
            }
        }
        di++;
     }
   }while(S->top > S->base);
   flag = 0;
}

void Show(int Maze[][L],int i,int j)
{
    for (int q = 0; q < L; q++)
    {
		for (int p = 0; p < L; p++)
		{
			if (((Maze[q][p] == ROUTE||Maze[q][p] == 2))&&(q!=i||p!=j)) {
				printf("  ");
			}
            else if(q==i&&p==j) { //显示当前的位置
                printf("十");
            }
			else{
				printf("回");
			}
		}
		printf("\n");
	}

}

void Show_Last(int Maze[][L])
{
    int i,j;
    for(i=0;i<L;i++){
        for(j=0;j<L;j++){
            if(Maze[i][j] == WALL){
                printf("回");
            }
            else if(Maze[i][j] == ROUTE || Maze[i][j] == 2){
                printf("  ");
            }
            else{
                printf("十");
            }
        }
        printf("\n");
    }
}

void Auto(int Maze[][L])
{
    Sqstack S;
    Box start,end;
    flag = Initstack(&S);
    if (flag == 0)  exit(1);
    start.x = 1;
    start.y = 1;
    start.di = 0;
    end.x = L-2;
    end.y = L-2;
    end.di = 0;
    MazePath(Maze, start, end,&S);
    if(flag == 0) printf("迷宫无入口到出口路径");
}

int main()
{
    srand((unsigned)time(NULL));
    int Maze[L][L],i,j,k;
    for(i=0;i<L;i++)
    {
        for(j=0;j<L;j++)
        {
            Maze[i][j]=WALL;
        }
    }
	for (i = 0; i < L; i++){
		Maze[i][0] = ROUTE;
		Maze[0][i] = ROUTE;
		Maze[i][L - 1] = ROUTE;
		Maze[L - 1][i] = ROUTE;
	}//数组初始化

    CreateMaze(Maze);//随机生成一个迷宫，不一定存在出口
    Maze[1][1] = ROUTE;//初始化迷宫入口

    Auto(Maze);
	return 0;
}
