# c-mini.c
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
#include <string.h>
/* ── Canvas dimensions ─────────────────────────────────────────────── */
#define CANVAS_ROWS 40
#define CANVAS_COLS 80
/* ── Shape types ───────────────────────────────────────────────────── */
#define SHAPE_CIRCLE    1
#define SHAPE_RECTANGLE 2
#define SHAPE_LINE      3
#define SHAPE_TRIANGLE  4
/* ── Maximum number of objects ─────────────────────────────────────── */
#define MAX_OBJECTS 50
/* ── Shape data structure ──────────────────────────────────────────── */
typedef struct {
    int type;       /* SHAPE_CIRCLE, SHAPE_RECTANGLE, ... */
    int active;     /* 1 = drawn, 0 = deleted              */
    int id;         /* unique object ID                     */
    /* Circle: centre (cx, cy), radius r                       */
    /* Rectangle: top-left (x1, y1), bottom-right (x2, y2)     */
    /* Line: endpoints (x1, y1) -> (x2, y2)                    */
    /* Triangle: three vertices (x1,y1), (x2,y2), (x3,y3)     */
    int x1, y1, x2, y2, x3, y3;
    int cx, cy, r;
} Shape;
/* ── Global state ──────────────────────────────────────────────────── */
char canvas[CANVAS_ROWS][CANVAS_COLS];
Shape objects[MAX_OBJECTS];
int object_count = 0;
int next_id      = 1;
/* ── Forward declarations ──────────────────────────────────────────── */
void clear_canvas(void);
void redraw_canvas(void);
void display_canvas(void);
void plot(int row, int col, char ch);
void draw_circle(int cy, int cx, int r);
void draw_rectangle(int y1, int x1, int y2, int x2);
void draw_line(int y1, int x1, int y2, int x2);
void draw_triangle(int y1, int x1, int y2, int x2, int y3, int x3);
void add_circle(void);
void add_rectangle(void);
void add_line(void);
void add_triangle(void);
void delete_object(void);
void list_objects(void);
void show_menu(void);
/* ══════════════════════════════════════════════════════════════════════
 *  Canvas helpers
 * ══════════════════════════════════════════════════════════════════════ */
/* Fill the canvas with the background character '_' */
void clear_canvas(void)
{
    int r, c;
    for (r = 0; r < CANVAS_ROWS; r++)
        for (c = 0; c < CANVAS_COLS; c++)
            canvas[r][c] = '_';
}
/* Place a character on the canvas (bounds-checked) */
void plot(int row, int col, char ch)
{
    if (row >= 0 && row < CANVAS_ROWS && col >= 0 && col < CANVAS_COLS)
        canvas[row][col] = ch;
}
/* Redraw every active object onto a fresh canvas */
void redraw_canvas(void)
{
    int i;
    clear_canvas();
    for (i = 0; i < object_count; i++) {
        if (!objects[i].active) continue;
        switch (objects[i].type) {
            case SHAPE_CIRCLE:
                draw_circle(objects[i].cy, objects[i].cx, objects[i].r);
                break;
            case SHAPE_RECTANGLE:
                draw_rectangle(objects[i].y1, objects[i].x1,
                               objects[i].y2, objects[i].x2);
                break;
            case SHAPE_LINE:
                draw_line(objects[i].y1, objects[i].x1,
                          objects[i].y2, objects[i].x2);
                break;
            case SHAPE_TRIANGLE:
                draw_triangle(objects[i].y1, objects[i].x1,
                              objects[i].y2, objects[i].x2,
                              objects[i].y3, objects[i].x3);
                break;
        }
    }
}
/* Print the canvas to stdout with a border */
void display_canvas(void)
{
    int r, c;
    redraw_canvas();
    /* Top border */
    printf("\n  +");
    for (c = 0; c < CANVAS_COLS; c++) printf("-");
    printf("+\n");
    /* Canvas rows */
    for (r = 0; r < CANVAS_ROWS; r++) {
        printf("  |");
        for (c = 0; c < CANVAS_COLS; c++)
            putchar(canvas[r][c]);
        printf("|\n");
    }
    /* Bottom border */
    printf("  +");
    for (c = 0; c < CANVAS_COLS; c++) printf("-");
    printf("+\n\n");
}
/* ══════════════════════════════════════════════════════════════════════
 *  Drawing functions
 * ══════════════════════════════════════════════════════════════════════ */
/* Draw a circle using the midpoint circle algorithm */
void draw_circle(int cy, int cx, int r)
{
    int x = 0, y = r;
    int d = 1 - r;
    while (x <= y) {
        /* Plot all 8 symmetric octant points */
        plot(cy + y, cx + x, '*');
        plot(cy + y, cx - x, '*');
        plot(cy - y, cx + x, '*');
        plot(cy - y, cx - x, '*');
        plot(cy + x, cx + y, '*');
        plot(cy + x, cx - y, '*');
        plot(cy - x, cx + y, '*');
        plot(cy - x, cx - y, '*');
        if (d < 0) {
            d += 2 * x + 3;
        } else {
            d += 2 * (x - y) + 5;
            y--;
        }
        x++;
    }
}
/* Draw an axis-aligned rectangle */
void draw_rectangle(int y1, int x1, int y2, int x2)
{
    int r, c;
    int top    = (y1 < y2) ? y1 : y2;
    int bottom = (y1 > y2) ? y1 : y2;
    int left   = (x1 < x2) ? x1 : x2;
    int right  = (x1 > x2) ? x1 : x2;
    /* Top and bottom edges */
    for (c = left; c <= right; c++) {
        plot(top, c, '*');
        plot(bottom, c, '*');
    }
    /* Left and right edges */
    for (r = top; r <= bottom; r++) {
        plot(r, left, '*');
        plot(r, right, '*');
    }
}
/* Draw a line using Bresenham's line algorithm */
void draw_line(int y1, int x1, int y2, int x2)
{
    int dx = abs(x2 - x1);
    int dy = abs(y2 - y1);
    int sx = (x1 < x2) ? 1 : -1;
    int sy = (y1 < y2) ? 1 : -1;
    int err = dx - dy;
    int e2;
    for (;;) {
        plot(y1, x1, '*');
        if (x1 == x2 && y1 == y2) break;
        e2 = 2 * err;
        if (e2 > -dy) { err -= dy; x1 += sx; }
        if (e2 <  dx) { err += dx; y1 += sy; }
    }
}
/* Draw a triangle (three lines connecting three vertices) */
void draw_triangle(int y1, int x1, int y2, int x2, int y3, int x3)
{
    draw_line(y1, x1, y2, x2);
    draw_line(y2, x2, y3, x3);
    draw_line(y3, x3, y1, x1);
}
/* ══════════════════════════════════════════════════════════════════════
 *  Object management (add / delete / list)
 * ══════════════════════════════════════════════════════════════════════ */
void add_circle(void)
{
    int cx, cy, r;
    if (object_count >= MAX_OBJECTS) {
        printf("  Error: maximum object limit (%d) reached.\n", MAX_OBJECTS);
        return;
    }
    printf("  Enter centre column (0-%d): ", CANVAS_COLS - 1);
    scanf("%d", &cx);
    printf("  Enter centre row    (0-%d): ", CANVAS_ROWS - 1);
    scanf("%d", &cy);
    printf("  Enter radius              : ");
    scanf("%d", &r);
    if (r <= 0) { printf("  Error: radius must be positive.\n"); return; }
    objects[object_count].type   = SHAPE_CIRCLE;
    objects[object_count].active = 1;
    objects[object_count].id     = next_id++;
    objects[object_count].cx     = cx;
    objects[object_count].cy     = cy;
    objects[object_count].r      = r;
    printf("  >> Circle added (ID %d)\n", objects[object_count].id);
    object_count++;
}
void add_rectangle(void)
{
    int x1, y1, x2, y2;
    if (object_count >= MAX_OBJECTS) {
        printf("  Error: maximum object limit (%d) reached.\n", MAX_OBJECTS);
        return;
    }
    printf("  Enter top-left  column (0-%d): ", CANVAS_COLS - 1);
    scanf("%d", &x1);
    printf("  Enter top-left  row    (0-%d): ", CANVAS_ROWS - 1);
    scanf("%d", &y1);
    printf("  Enter bottom-right column (0-%d): ", CANVAS_COLS - 1);
    scanf("%d", &x2);
    printf("  Enter bottom-right row    (0-%d): ", CANVAS_ROWS - 1);
    scanf("%d", &y2);
    objects[object_count].type   = SHAPE_RECTANGLE;
    objects[object_count].active = 1;
    objects[object_count].id     = next_id++;
    objects[object_count].x1     = x1;
    objects[object_count].y1     = y1;
    objects[object_count].x2     = x2;
    objects[object_count].y2     = y2;
    printf("  >> Rectangle added (ID %d)\n", objects[object_count].id);
    object_count++;
}
void add_line(void)
{
    int x1, y1, x2, y2;
    if (object_count >= MAX_OBJECTS) {
        printf("  Error: maximum object limit (%d) reached.\n", MAX_OBJECTS);
        return;
    }
    printf("  Enter start column (0-%d): ", CANVAS_COLS - 1);
    scanf("%d", &x1);
    printf("  Enter start row    (0-%d): ", CANVAS_ROWS - 1);
    scanf("%d", &y1);
    printf("  Enter end   column (0-%d): ", CANVAS_COLS - 1);
    scanf("%d", &x2);
    printf("  Enter end   row    (0-%d): ", CANVAS_ROWS - 1);
    scanf("%d", &y2);
    objects[object_count].type   = SHAPE_LINE;
    objects[object_count].active = 1;
    objects[object_count].id     = next_id++;
    objects[object_count].x1     = x1;
    objects[object_count].y1     = y1;
    objects[object_count].x2     = x2;
    objects[object_count].y2     = y2;
    printf("  >> Line added (ID %d)\n", objects[object_count].id);
    object_count++;
}
void add_triangle(void)
{
    int x1, y1, x2, y2, x3, y3;
    if (object_count >= MAX_OBJECTS) {
        printf("  Error: maximum object limit (%d) reached.\n", MAX_OBJECTS);
        return;
    }
    printf("  Enter vertex-1 column (0-%d): ", CANVAS_COLS - 1);
    scanf("%d", &x1);
    printf("  Enter vertex-1 row    (0-%d): ", CANVAS_ROWS - 1);
    scanf("%d", &y1);
    printf("  Enter vertex-2 column (0-%d): ", CANVAS_COLS - 1);
    scanf("%d", &x2);
    printf("  Enter vertex-2 row    (0-%d): ", CANVAS_ROWS - 1);
    scanf("%d", &y2);
    printf("  Enter vertex-3 column (0-%d): ", CANVAS_COLS - 1);
    scanf("%d", &x3);
    printf("  Enter vertex-3 row    (0-%d): ", CANVAS_ROWS - 1);
    scanf("%d", &y3);
    objects[object_count].type   = SHAPE_TRIANGLE;
    objects[object_count].active = 1;
    objects[object_count].id     = next_id++;
    objects[object_count].x1     = x1;
    objects[object_count].y1     = y1;
    objects[object_count].x2     = x2;
    objects[object_count].y2     = y2;
    objects[object_count].x3     = x3;
    objects[object_count].y3     = y3;
    printf("  >> Triangle added (ID %d)\n", objects[object_count].id);
    object_count++;
}
void delete_object(void)
{
    int id, i, found = 0;
    list_objects();
    printf("  Enter object ID to delete: ");
    scanf("%d", &id);
    for (i = 0; i < object_count; i++) {
        if (objects[i].id == id && objects[i].active) {
            objects[i].active = 0;
            found = 1;
            printf("  >> Object %d deleted.\n", id);
            break;
        }
    }
    if (!found)
        printf("  Error: no active object with ID %d.\n", id);
}
/* Print a table of all active objects */
void list_objects(void)
{
    int i, any = 0;
    printf("\n  %-4s  %-12s  %s\n", "ID", "Type", "Parameters");
    printf("  ----  ------------  -----------------------------------------\n");
    for (i = 0; i < object_count; i++) {
        if (!objects[i].active) continue;
        any = 1;
        switch (objects[i].type) {
            case SHAPE_CIRCLE:
                printf("  %-4d  %-12s  centre=(%d,%d)  radius=%d\n",
                       objects[i].id, "Circle",
                       objects[i].cx, objects[i].cy, objects[i].r);
                break;
            case SHAPE_RECTANGLE:
                printf("  %-4d  %-12s  top-left=(%d,%d)  bottom-right=(%d,%d)\n",
                       objects[i].id, "Rectangle",
                       objects[i].x1, objects[i].y1,
                       objects[i].x2, objects[i].y2);
                break;
            case SHAPE_LINE:
                printf("  %-4d  %-12s  from=(%d,%d)  to=(%d,%d)\n",
                       objects[i].id, "Line",
                       objects[i].x1, objects[i].y1,
                       objects[i].x2, objects[i].y2);
                break;
            case SHAPE_TRIANGLE:
                printf("  %-4d  %-12s  v1=(%d,%d)  v2=(%d,%d)  v3=(%d,%d)\n",
                       objects[i].id, "Triangle",
                       objects[i].x1, objects[i].y1,
                       objects[i].x2, objects[i].y2,
                       objects[i].x3, objects[i].y3);
                break;
        }
    }
    if (!any)
        printf("  (no objects on canvas)\n");
    printf("\n");
}
/* ══════════════════════════════════════════════════════════════════════
 *  Menu
 * ══════════════════════════════════════════════════════════════════════ */
void show_menu(void)
{
    printf("  ╔═══════════════════════════════════════╗\n");
    printf("  ║      2D GRAPHICS EDITOR               ║\n");
    printf("  ╠═══════════════════════════════════════╣\n");
    printf("  ║  1. Add Circle                        ║\n");
    printf("  ║  2. Add Rectangle                     ║\n");
    printf("  ║  3. Add Line                          ║\n");
    printf("  ║  4. Add Triangle                      ║\n");
    printf("  ║  5. Delete Object                     ║\n");
    printf("  ║  6. List Objects                      ║\n");
    printf("  ║  7. Display Picture                   ║\n");
    printf("  ║  8. Clear All                         ║\n");
    printf("  ║  0. Exit                              ║\n");
    printf("  ╚═══════════════════════════════════════╝\n");
    printf("  Enter choice: ");
}
/* ══════════════════════════════════════════════════════════════════════
 *  Main
 * ══════════════════════════════════════════════════════════════════════ */
int main(void)
{
    int choice;
    clear_canvas();
    printf("\n  Welcome to the 2D Graphics Editor!\n");
    printf("  Canvas size: %d rows x %d cols\n", CANVAS_ROWS, CANVAS_COLS);
    printf("  Shapes are drawn with '*', background is '_'.\n\n");
    do {
        show_menu();
        scanf("%d", &choice);
        printf("\n");
        switch (choice) {
            case 1: add_circle();    break;
            case 2: add_rectangle(); break;
            case 3: add_line();      break;
            case 4: add_triangle();  break;
            case 5: delete_object(); break;
            case 6: list_objects();  break;
            case 7: display_canvas();break;
            case 8:
                object_count = 0;
                next_id = 1;
                clear_canvas();
                printf("  >> Canvas cleared.\n\n");
                break;
            case 0:
                printf("  Goodbye!\n");
                break;
            default:
                printf("  Invalid choice. Try again.\n\n");
        }
    } while (choice != 0);
    return 0;
}
