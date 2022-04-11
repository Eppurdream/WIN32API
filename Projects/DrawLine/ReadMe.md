# Win32를 이용해 선을 그리는 프로그램 작성함

- 코드
<details>

```cpp
// GDIPJ.cpp : 애플리케이션에 대한 진입점을 정의합니다.
//

// GDIPJ.cpp : 애플리케이션에 대한 진입점을 정의합니다.
//
#include <vector>
using namespace std;

#include "framework.h"
#include "GDIPJ.h"
#include "PointPair.h"

#define MAX_LOADSTRING 100

#define SCREENWIDTH 1920
#define SCREENHEIGTH 1080

// 전역 변수:
HINSTANCE hInst;                                // 현재 인스턴스입니다.
WCHAR szTitle[MAX_LOADSTRING];                  // 제목 표시줄 텍스트입니다.
WCHAR szWindowClass[MAX_LOADSTRING];            // 기본 창 클래스 이름입니다.

vector<PointPair> ptList;
vector<PointPair> redoPtList;

// 이 코드 모듈에 포함된 함수의 선언을 전달합니다:
ATOM                MyRegisterClass(HINSTANCE hInstance);
BOOL                InitInstance(HINSTANCE, int);
LRESULT CALLBACK    WndProc(HWND, UINT, WPARAM, LPARAM);
INT_PTR CALLBACK    About(HWND, UINT, WPARAM, LPARAM);

int APIENTRY wWinMain(_In_ HINSTANCE hInstance,
                     _In_opt_ HINSTANCE hPrevInstance,
                     _In_ LPWSTR    lpCmdLine,
                     _In_ int       nCmdShow)
{
    UNREFERENCED_PARAMETER(hPrevInstance);
    UNREFERENCED_PARAMETER(lpCmdLine);

    LoadStringW(hInstance, IDS_APP_TITLE, szTitle, MAX_LOADSTRING);
    LoadStringW(hInstance, IDC_GDIPJ, szWindowClass, MAX_LOADSTRING);
    MyRegisterClass(hInstance);

    // 애플리케이션 초기화를 수행합니다:
    if (!InitInstance (hInstance, nCmdShow))
    {
        return FALSE;
    }

    HACCEL hAccelTable = LoadAccelerators(hInstance, MAKEINTRESOURCE(IDC_GDIPJ));

    MSG msg;

    // 기본 메시지 루프입니다:
    while (GetMessage(&msg, nullptr, 0, 0))
    {
        if (!TranslateAccelerator(msg.hwnd, hAccelTable, &msg))
        {
            TranslateMessage(&msg);
            DispatchMessage(&msg);
        }
    }

    return (int) msg.wParam;
}

ATOM MyRegisterClass(HINSTANCE hInstance)
{
    WNDCLASSEXW wcex;

    wcex.cbSize = sizeof(WNDCLASSEX);

    wcex.style          = CS_HREDRAW | CS_VREDRAW | CS_DBLCLKS;
    wcex.lpfnWndProc    = WndProc;
    wcex.cbClsExtra     = 0;
    wcex.cbWndExtra     = 0;
    wcex.hInstance      = hInstance;
    wcex.hIcon          = LoadIcon(hInstance, MAKEINTRESOURCE(IDI_GDIPJ));
    wcex.hCursor        = LoadCursor(nullptr, IDC_ARROW);
    wcex.hbrBackground  = (HBRUSH)(COLOR_WINDOW+1);
    wcex.lpszMenuName   = NULL;
    wcex.lpszClassName  = szWindowClass;
    wcex.hIconSm        = LoadIcon(wcex.hInstance, MAKEINTRESOURCE(IDI_SMALL));

    return RegisterClassExW(&wcex);
}

BOOL InitInstance(HINSTANCE hInstance, int nCmdShow)
{
   hInst = hInstance; // 인스턴스 핸들을 전역 변수에 저장합니다.

   HWND hWnd = CreateWindowW(szWindowClass, szTitle, WS_OVERLAPPEDWINDOW,
      CW_USEDEFAULT, 0, SCREENWIDTH, SCREENHEIGTH, nullptr, nullptr, hInstance, nullptr);

   if (!hWnd)
   {
      return FALSE;
   }

   ShowWindow(hWnd, nCmdShow);
   UpdateWindow(hWnd);

   return TRUE;
}

// WM_KEYDOWN, WM_KEYUP => alt, F10 하고 시스템키는 감지 안함

#define M 5
//#define R 5
//#define PENWIDTH 2

// 기존에 그려지니 선이 지워지지 않게
// 창이 줄어들거나 아래로 내려갔다와도 기존의 그림이 남아있게 (최근 5개까지만)

POINT beforePoint[5][2];
int _currentIndex = 0;

LRESULT CALLBACK WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
{
    static HPEN hBluePen;
    static BOOL bDraw = FALSE;
    static POINT ptStart, ptEnd;

    switch (message)
    {
    case WM_CREATE:
        hBluePen = CreatePen(PS_DASH, 1, RGB(0, 0, 255));

        break;
    case WM_LBUTTONDOWN:
        ptStart.x = LOWORD(lParam);
        ptStart.y = HIWORD(lParam);
        ptEnd = ptStart;

        break;
    case WM_LBUTTONUP:
    {
        bDraw = TRUE;
        InvalidateRect(hWnd, NULL, FALSE);

        PointPair pt(ptStart, ptEnd);

        ptList.push_back(pt);

        if (redoPtList.size() != 0)
        {
            redoPtList.clear();
        }
        break;

    }
    case WM_KEYDOWN:
    {
        switch (wParam)
        {
        case 0x5A:
        {
            if (ptList.size() >= 1)
            {
                redoPtList.push_back(*(ptList.end() - 1));
                ptList.pop_back();
                
                InvalidateRect(hWnd, NULL, TRUE);
            }
            break;
        }
        case 0x52:
        {
            if (redoPtList.size() >= 1)
            {
                ptList.push_back(*(redoPtList.end() - 1));
                redoPtList.pop_back();
                InvalidateRect(hWnd, NULL, TRUE);
            }
            break;
        }
        }
        break;
    }
    case WM_MOUSEMOVE:
        if (wParam & MK_LBUTTON)
        {
            HDC hdc = GetDC(hWnd);
            // #1 이전 라인을 삭제하는 모드
            int oldMode = SetROP2(hdc, R2_NOTXORPEN); // 화면색상과 XOR 지우기
            HPEN hOldPen = static_cast<HPEN>(SelectObject(hdc, hBluePen));

            MoveToEx(hdc, ptStart.x, ptStart.y, NULL);
            LineTo(hdc, ptEnd.x, ptEnd.y);
            // #2 새로운 라인을 그리는 모드

            ptEnd.x = LOWORD(lParam);
            ptEnd.y = HIWORD(lParam);

            MoveToEx(hdc, ptStart.x, ptStart.y, NULL);
            LineTo(hdc, ptEnd.x, ptEnd.y);

            SelectObject(hdc, hOldPen);
            SetROP2(hdc, oldMode);
            ReleaseDC(hWnd, hdc);
        }
        break;
    case WM_PAINT:
        {
            PAINTSTRUCT ps;
            HDC hdc = BeginPaint(hWnd, &ps);

            for (auto it = ptList.begin(); it != ptList.end(); ++it)
            {
                MoveToEx(hdc, it->start.x, it->start.y, NULL);
                LineTo(hdc, it->end.x, it->end.y);

                Rectangle(hdc, it->start.x - M, it->start.y - M, it->start.x + M, it->start.y + M);
                Rectangle(hdc, it->start.x - M, it->start.y - M, it->start.x + M, it->start.y + M);

                bDraw = FALSE;
            }

            EndPaint(hWnd, &ps);
        }
        break;
    case WM_DESTROY:

        DeleteObject(hBluePen);

        PostQuitMessage(0);
        break;
    default:

        return DefWindowProc(hWnd, message, wParam, lParam);
    }
    return 0;
}
```

</details>
