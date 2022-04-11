# 시간에 따라 움직이는 시계를 만든 프로젝트

<detail>


```cpp
// GDIPJ.cpp : 애플리케이션에 대한 진입점을 정의합니다.
//
using namespace std;

#include "framework.h"
#include "GDIPJ.h"

#define _USE_MATH_DEFINES
#include <math.h>

#define R 100
#define RADIAN(X) ((X) * M_PI / 180)

// X degree를 라디안값으로 변경하는 매크로

#define MAX_LOADSTRING 100

#define SCREENWIDTH 960
#define SCREENHEIGTH 540

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

   SendMessage(hWnd, WM_TIMER, 1, NULL);

   return TRUE;
}

// WM_KEYDOWN, WM_KEYUP => alt, F10 하고 시스템키는 감지 안함

// 기존에 그려지니 선이 지워지지 않게
// 창이 줄어들거나 아래로 내려갔다와도 기존의 그림이 남아있게 (최근 5개까지만)

// SetTimer, KillTimer
#define STEP 10
#define R 100

RECT rtClient; // /화면 영역을 저장할 RECT

LRESULT CALLBACK WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
{
    SYSTEMTIME st;

    double theta;
    POINT pt;

    static RECT rtClient;
    static TCHAR szTime[120]; // 시간 문자열 저장용
    static TCHAR msg[120];    // 이건 사용자 메시지 저장용
    static double sec, min, hour;
    static HPEN hPens[3]; // 바늘을 그릴 펜 색상
    static POINT ptCircle;

    switch (message)
    {
    case WM_CREATE:
    {
        hPens[0] = CreatePen(PS_SOLID, 8, RGB(0, 0, 255));
        hPens[1] = CreatePen(PS_SOLID, 8, RGB(0, 255, 0));
        hPens[2] = CreatePen(PS_SOLID, 8, RGB(255, 0, 0));

        SetTimer(hWnd, 1, 1000, NULL);
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
        GetLocalTime(&st); // 현재 시간을 참조해서 구조체에 정보르르 실어준다.
        wsprintf(szTime, _T("%d-%d-%d : %d:%d:%d"),
            st.wYear, st.wMonth, st.wDay,
            st.wHour, st.wMinute, st.wSecond);
        InvalidateRect(hWnd, NULL, TRUE);

        sec = st.wSecond;
        min = st.wMinute + st.wSecond / 60.0;
        hour = st.wHour + min / 60.0;

        break;
    }
    case WM_PAINT:
        {
            PAINTSTRUCT ps;
            HDC hdc = BeginPaint(hWnd, &ps);

            DrawText(hdc, szTime, -1, &rtClient, DT_CENTER);

            Ellipse(hdc, ptCircle.x - R, ptCircle.y - R, ptCircle.x + R, ptCircle.y + R);

            for (int i = 1; i <= 12; i++)
            {
                theta = ((double)i - 3) * 30; // 여기에 i에 맞는 각도가 나와야 함
                pt.x = cos(RADIAN(theta)) * R + ptCircle.x + 0.5; // cos 함수를 이용해서 x위치를 구해야 함
                pt.y = sin(RADIAN(theta)) * R + ptCircle.y + 0.5; // sin 함수를 이용해서 구해야 함

                TCHAR str[10];
                wsprintf(str, _T("%d"), i);

                RECT rt;
                SetRect(&rt, pt.x - 10, pt.y - 10, pt.x + 10, pt.y + 10);
                DrawText(hdc, str, -1, &rt, DT_CENTER);

                // 여기서 str를 pt 위치에 출력하면 됨
            }

            // 시침
            theta = (hour - 15) * 15; // 현재 hour값에 비례한 theta를 찾아야 함
            pt.x = cos(RADIAN(theta)) * (R - 40) + ptCircle.x + 0.5; // 각각 알맞은
            pt.y = sin(RADIAN(theta)) * (R - 40) + ptCircle.y + 0.5;

            HPEN hOldPen = static_cast<HPEN>(SelectObject(hdc, hPens[0]));

            MoveToEx(hdc, ptCircle.x, ptCircle.y, NULL);
            LineTo(hdc, pt.x, pt.y);
            // 분침
            theta = (min - 15) * 6; // 현재 hour값에 비례한 theta를 찾아야 함
            pt.x = cos(RADIAN(theta)) * (R - 20) + ptCircle.x + 0.5; // 각각 알맞은
            pt.y = sin(RADIAN(theta)) * (R - 20) + ptCircle.y + 0.5;

            hOldPen = static_cast<HPEN>(SelectObject(hdc, hPens[1]));

            MoveToEx(hdc, ptCircle.x, ptCircle.y, NULL);
            LineTo(hdc, pt.x, pt.y);
            // 초침
            theta = (sec - 15) * 6; // 현재 hour값에 비례한 theta를 찾아야 함
            pt.x = cos(RADIAN(theta)) * (R - 5) + ptCircle.x + 0.5; // 각각 알맞은
            pt.y = sin(RADIAN(theta)) * (R - 5) + ptCircle.y + 0.5;

            hOldPen = static_cast<HPEN>(SelectObject(hdc, hPens[2]));

            MoveToEx(hdc, ptCircle.x, ptCircle.y, NULL);
            LineTo(hdc, pt.x, pt.y);


            EndPaint(hWnd, &ps);
        }
        break;
    case WM_DESTROY:
        KillTimer(hWnd, 1);

        for (int i = 0; i < 3; i++)
        {
            DeleteObject(hPens[i]);
        }

        PostQuitMessage(0);
        break;
    default:

        return DefWindowProc(hWnd, message, wParam, lParam);
    }
    return 0;
}
```


</detail>