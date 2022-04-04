# Win32를 이용해 선을 그리는 프로그램 작성함

- 코드
<details>

```cpp
// GDIPJ.cpp : 애플리케이션에 대한 진입점을 정의합니다.
//

#include "framework.h"
#include "GDIPJ.h"

#define MAX_LOADSTRING 100

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
      CW_USEDEFAULT, 0, 800, 600, nullptr, nullptr, hInstance, nullptr);

   if (!hWnd)
   {
      return FALSE;
   }

   ShowWindow(hWnd, nCmdShow);
   UpdateWindow(hWnd);

   return TRUE;
}

// WM_KEYDOWN, WM_KEYUP => alt, F10 하고 시스템키는 감지 안함

#define R 5
#define PENWIDTH 2

LRESULT CALLBACK WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
{
    static POINT FPoint;
    static POINT LPoint;

    static HPEN hDownPen;
    static HPEN hUpPen;
    static HPEN hPen;

    // GDI 객체, 브러시, 펜 Create[xxx]
    // SelectObject, DeleteObject
    switch (message)
    {
    case WM_CREATE:
        hDownPen = CreatePen(PS_DASH, 1, RGB(0, 0, 0));
        hUpPen = CreatePen(PS_SOLID, PENWIDTH, RGB(0, 0, 0));

        hPen = hDownPen;

        break;
    case WM_LBUTTONDOWN:
        hPen = hDownPen;
        FPoint.x = LOWORD(lParam);
        FPoint.y = HIWORD(lParam);

        LPoint.x = LOWORD(lParam);
        LPoint.y = HIWORD(lParam);

        InvalidateRect(hWnd, NULL, TRUE);
        break;
    case WM_MOUSEMOVE:
        if (GetAsyncKeyState(VK_LBUTTON) & 0x8000)
        {
            LPoint.x = LOWORD(lParam);
            LPoint.y = HIWORD(lParam);

            InvalidateRect(hWnd, NULL, TRUE);
        }
        break;
    case WM_LBUTTONUP:
        hPen = hUpPen;
        InvalidateRect(hWnd, NULL, TRUE);
        break;
    case WM_PAINT:
        {
            PAINTSTRUCT ps;
            HDC hdc = BeginPaint(hWnd, &ps);

            SelectObject(hdc, hPen);
            MoveToEx(hdc, FPoint.x, FPoint.y, NULL);
            LineTo(hdc, LPoint.x, LPoint.y);

            if (hPen == hUpPen)
            {
                Ellipse(hdc, FPoint.x - R, FPoint.y - R, FPoint.x + R, FPoint.y + R);
                Ellipse(hdc, LPoint.x - R, LPoint.y - R, LPoint.x + R, LPoint.y + R);
            }

            EndPaint(hWnd, &ps);
        }
        break;
    case WM_DESTROY:

        DeleteObject(hDownPen);
        DeleteObject(hUpPen);
        DeleteObject(hPen);

        PostQuitMessage(0);
        break;
    default:
        return DefWindowProc(hWnd, message, wParam, lParam);
    }
    return 0;
}
```

</details>