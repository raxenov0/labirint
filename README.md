
#include <iostream>
#include <ctime>
#include <windows.h> 
#include <vector>

#define _WIN_32_WINNT 0x0A00

using namespace std;

struct road {
    int x;
    int y;
    bool top = true;
    bool right = true;
    bool bottom = true;
    bool left = true;
    bool visited = false;
    bool stop = false;
    bool start = false;
    bool end = false;
    bool paint = false;
    };

void Generate(int, int);

road** pointS;

bool check_cell(int i,int j, int width , int height) {
    if (j< 0 || j> width - 1 || i< 0 || i > height - 1) {
        return false;
    }
    return true;
}
road check_neighbors(road **arr, int i, int j, int width,int height) {
    road neighbors[4];
    int count = 0;
    int top = check_cell(i - 1, j, width, height);
    int right = check_cell(i, j + 1, width, height);
    int bottom = check_cell(i+1, j , width, height);
    int left = check_cell(i, j - 1, width, height);

    if ((top == 1) && (arr[i - 1][j].visited == 0)) {
        neighbors[count] = arr[i - 1][j];
        count++;
    }
    if ((right == 1) && (arr[i][j+1].visited == 0) ) {
        neighbors[count] = arr[i][j + 1];
        count++;
    }
    if ((bottom == 1) && (arr[i + 1][j].visited == 0)) {
        neighbors[count] = arr[i + 1][j];
        count++;
    }
    if ((left == 1) && (arr[i][j - 1].visited == 0)) {
        neighbors[count] = arr[i][j - 1];
        count++;
    }
    if (count == 0) {
        arr[i][j].stop = true;
        return arr[i][j];
    }
   
    return neighbors[rand() % count];
}

void remove_walls(road **arr,road current_cell, road next_cell) {
    int dx = current_cell.x - next_cell.x;
    if (dx == 1) {
        arr[current_cell.y][current_cell.x].left = false;
        arr[next_cell.y][next_cell.x].right = false;
        current_cell.left = false;
        next_cell.right = false;
    }
    else if (dx == -1) {
        arr[current_cell.y][current_cell.x].right = false;
        arr[next_cell.y][next_cell.x].left = false;
        current_cell.right = false;
        next_cell.left = false;
    }
    int dy = current_cell.y - next_cell.y;
    if (dy == 1) {
        arr[current_cell.y][current_cell.x].top = false;
        arr[next_cell.y][next_cell.x].bottom = false;
        current_cell.top = false;
        next_cell.bottom = false;
    }
    else if (dy == -1) {
        arr[current_cell.y][current_cell.x].bottom = false;
        arr[next_cell.y][next_cell.x].top = false;
        current_cell.bottom = false;
        next_cell.top = false;
    }
}


 void Generate(int width, int height) {
    srand(time(NULL));
    road stop;
    stop.stop = true;
    road** labir = new road* [width];
    for (int i = 0; i < width; i++) {
        labir[i] = new road[height];
    }
    for (int i = 0; i < width; i++) {
        for (int j = 0; j < height; j++) {
            labir[i][j].x = j;
            labir[i][j].y = i;
        }
    }
    road current_cell = labir[0][0];
    road next_cell;
    vector <road> stack;
    int temp = 0;
    while (true) {
        labir[current_cell.y][current_cell.x].visited = true;
        current_cell.visited = true;
        int i = current_cell.y;
        int j = current_cell.x;  
        if (check_neighbors(labir, i, j, width, height).stop == true) {
            current_cell = stack[stack.size()-1];
            stack.pop_back();
        }
        else {
            next_cell = check_neighbors(labir, i, j, width, height);
            labir[next_cell.y][next_cell.x].visited = true;
            next_cell.visited = true;
            stack.push_back(labir[current_cell.y][current_cell.x]);
            remove_walls(labir,current_cell, next_cell);
            current_cell = next_cell;
            temp++;
        }
        if (temp == width * height -1) break;
        
    }
    pointS = labir;
}

 road check_walls(road** arr, int i, int j) {
     road neighbors[4];
     int count = 0;
     if ((arr[i][j].top == 0) && (arr[i - 1][j].visited == 0)) {
         neighbors[count] = arr[i - 1][j];
         count++;
     }
     if ((arr[i][j].right == 0) && (arr[i][j + 1].visited == 0)) {
         neighbors[count] = arr[i][j + 1];
         count++;
     }
     if ((arr[i][j].bottom == 0) && (arr[i + 1][j].visited == 0)) {
         neighbors[count] = arr[i + 1][j];
         count++;
     }
     if ((arr[i][j].left == 0) && (arr[i][j - 1].visited == 0)) {
         neighbors[count] = arr[i][j - 1];
         count++;
     }
     if (count == 0) {
         arr[i][j].stop = true;
         return arr[i][j];
     }
     return neighbors[rand() % count];
 }

 void View(int width,int *start,int *end) {
    HWND hwnd = GetConsoleWindow();
    HDC dc = GetDC(hwnd);
    SelectObject(dc, GetStockObject(DC_BRUSH));
    SetDCBrushColor(dc, RGB(245, 245, 237));
    SelectObject(dc, GetStockObject(DC_PEN));
    SetDCPenColor(dc, RGB(26, 25, 23));
    Rectangle(dc, 0, 50, 400, 450);
    int scrW = 400 / width;
    int scrH = 400 / width;
    for (int i = 0; i < width; i++) {
        for (int j = 0; j < width; j++) {
            int x = scrW * j;
            int y = scrH * i;

            SelectObject(dc, GetStockObject(DC_PEN));
            SetDCPenColor(dc, RGB(245, 245, 237));
            Rectangle(dc, x, 50 + y, x + scrW, 50 + y + scrH);
        }
    }
    for (int i = 0; i < width; i++) {
        for (int j = 0; j < width; j++) {
            int x = scrW * j;
            int y = scrH * i;
            if (pointS[i][j].visited) {
                if (pointS[i][j].top) {
                    SelectObject(dc, GetStockObject(DC_PEN));
                    SetDCPenColor(dc, RGB(46, 217, 120));
                    MoveToEx(dc, x, 50 + y, 0);
                    LineTo(dc, x + scrW, 50 + y);
                }
                if (pointS[i][j].bottom) {
                    SelectObject(dc, GetStockObject(DC_PEN));
                    SetDCPenColor(dc, RGB(46, 217, 120));
                    MoveToEx(dc, x, 50 + y + scrH, 0);
                    LineTo(dc, x + scrW, 50 + y + scrH);
                }
                if (pointS[i][j].left) {
                    SelectObject(dc, GetStockObject(DC_PEN));
                    SetDCPenColor(dc, RGB(46, 217, 120));
                    MoveToEx(dc, x, 50 + y, 0);
                    LineTo(dc, x, 50 + y + scrH);
                }
                if (pointS[i][j].right) {
                    SelectObject(dc, GetStockObject(DC_PEN));
                    SetDCPenColor(dc, RGB(46, 217, 120));
                    MoveToEx(dc, x + scrW, 50 + y, 0);
                    LineTo(dc, x + scrW, 50 + y + scrH);
                }

            }
            else {
                SelectObject(dc, GetStockObject(DC_PEN));
                SetDCPenColor(dc, RGB(232, 229, 37));
                Rectangle(dc, x, 50 + y, x + scrW, 50 + y + scrH);
            }

        }
    }
    SelectObject(dc, GetStockObject(DC_BRUSH));
    SetDCBrushColor(dc, RGB(180, 46, 217));
    SelectObject(dc, GetStockObject(DC_PEN));
    SetDCPenColor(dc, RGB(180, 46, 217));
    pointS[start[0]][start[1]].start = true;
    pointS[end[0] - 1][end[1] - 1].end = true;
    Rectangle(dc, scrW*start[0]+2, scrH*start[1]+52, scrH*(start[0]+1)-2, scrH*(start[1]+1)+48);
    Rectangle(dc, 2+(end[0]-1)*scrW, 52+scrH*(end[1]-1), scrW*end[0]-2, 48+scrH*end[1]);
 }
void GenerateAndView(int width,int* start, int* end) {
    Generate(width, width);
    View(width,start,end);
}

void FoundWay(road** arr, int width, int *start ,int* end) {
    road current_cell = arr[start[0]][start[1]];
    road next_cell;
    vector <road> stack;
    int temp = 0;
    for (int i = 0; i < width; i++) {
        for (int j = 0; j < width; j++) {
            arr[i][j].visited = false;
            arr[i][j].stop = false;
        }
    }
    while (true) {
        arr[current_cell.y][current_cell.x].visited = true;
        current_cell.visited = true;
        int i = current_cell.y;
        int j = current_cell.x;
        if (check_walls(arr, i, j).stop == true) {
            current_cell = stack[stack.size() - 1];
            stack.pop_back();
            arr[current_cell.y][current_cell.x].paint = false;
        }
        else {
            next_cell = check_walls(arr, i, j);
            arr[next_cell.y][next_cell.x].visited = true;
            next_cell.visited = true;
            stack.push_back(arr[current_cell.y][current_cell.x]);
            arr[current_cell.y][current_cell.x].paint = true;
            current_cell = next_cell;
        }
        if (current_cell.x == (end[1]-1) && current_cell.y == (end[0]-1)) {
            break;
        }

    }
    HWND hwnd = GetConsoleWindow();
    HDC dc = GetDC(hwnd);
    for (int i = 0; i < width; i++) {
        for (int j = 0; j < width; j++) {
            int scrW = 400 / width;
            int scrH = 400 / width;
            int x = scrW * j;
            int y = scrH * i;
            if (pointS[i][j].visited && pointS[i][j].paint) {
                SelectObject(dc, GetStockObject(DC_BRUSH));
                SetDCBrushColor(dc, RGB(180, 46, 217));
                SelectObject(dc, GetStockObject(DC_PEN));
                SetDCPenColor(dc, RGB(180, 46, 217));
                Ellipse(dc, scrW * j + 8, scrH * i + 58, scrH * (j + 1) - 8, scrH * (i + 1) + 42);
            }
        }
    }
}

int main() {
    int width = 20;
    int start[] = { 5,5 };
    int end[] = { width-1 ,width-1 };
    GenerateAndView(width,start,end);
    FoundWay(pointS, width, start,end);
}
