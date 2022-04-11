# 움직이는 원을 win32로 만든 프로젝트

- 코드

<details>


```cpp
// GDIPJ.cpp : 애플리케이션에 대한 진입점을 정의합니다.
//
using namespace std;

#include "framework.h"
#include "GDIPJ.h"

#define MAX_LOADSTRING 100

#define SCREENWIDTH 1920
#define SCREENHEIGTH 1080

// 전역 변수:
HINSTANCE hInst;                                // 현재 인스턴스입니다.
WCHAR szTitle[MAX_LOADSTRING];                  // 제목 표시줄 텍스트입니다.
WCHAR szWindowClass[MAX_LOADSTRING];            // 기본 창 클래스 이름입니다.

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

// 기존에 그려지니 선이 지워지지 않게
// 창이 줄어들거나 아래로 내려갔다와도 기존의 그림이 남아있게 (최근 5개까지만)

// SetTimer, KillTimer
#define STEP 10
#define R 20

POINT ptCircle;
RECT rtClient; // /화면 영역을 저장할 RECT
int direction = VK_RIGHT;

void TimerProc(HWND hWnd, UINT uMsg, UINT_PTR nID, DWORD dwTime)
{
    switch (direction)
    {
    case VK_LEFT:
        ptCircle.x -= STEP;
        break;
    case VK_RIGHT:
        ptCircle.x += STEP;
        break;
    case VK_DOWN:
        ptCircle.y += STEP;
        break;
    case VK_UP:
        ptCircle.y -= STEP;
        break;
    }

    if (ptCircle.x - R <= 0)
    {
        ptCircle.x = R;
        direction = VK_RIGHT;
    }

    if (ptCircle.y - R <= 0)
    {
        ptCircle.y = R;
        direction = VK_DOWN;
    }

    if (ptCircle.x + R >= rtClient.right)
    {
        ptCircle.x = rtClient.right - R;
        direction = VK_LEFT;
    }

    if (ptCircle.y + R >= rtClient.right)
    {
        ptCircle.y = rtClient.bottom - R;
        direction = VK_UP;
    }

    InvalidateRect(hWnd, &rtClient, TRUE);
}

LRESULT CALLBACK WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
{

    switch (message)
    {
    case WM_CREATE:
    {
        SetTimer(hWnd, 1, 50, (TIMERPROC)(TimerProc));
        // 마지막 타이머 프로시저가 NULL이면 주어진 시간마다 Wnd로 WM_TIMER라는 메시지를 실행
        break;
    }
    case WM_KEYDOWN:
    {
        switch (wParam)
        {
        case VK_LEFT:
            direction = VK_LEFT;
            break;
        case VK_RIGHT:
            direction = VK_RIGHT;
            break;
        case VK_DOWN:
            direction = VK_DOWN;
            break;
        case VK_UP:
            direction = VK_UP;
            break;
        }
        break;
    }
    case WM_SIZE:
    {
        GetClientRect(hWnd, &rtClient);
        ptCircle.x = rtClient.right / 2;
        ptCircle.y = rtClient.bottom / 2;
        break;
    }
    case WM_TIMER:
    {
        break;
    }
    case WM_PAINT:
        {
            PAINTSTRUCT ps;
            HDC hdc = BeginPaint(hWnd, &ps);

            Ellipse(hdc, ptCircle.x - 8, ptCircle.y - 8, ptCircle.x + 8, ptCircle.y + 8);

            EndPaint(hWnd, &ps);
        }
        break;
    case WM_DESTROY:
        KillTimer(hWnd, 1);

        PostQuitMessage(0);
        break;
    default:

        return DefWindowProc(hWnd, message, wParam, lParam);
    }
    return 0;
}
```

</details>
