#include <stdio.h>
#include <stdlib.h>
#include <math.h>

#define WIDTH 80
#define HEIGHT 24
#define MAX 100

char canvas[HEIGHT][WIDTH];

struct shape {
    int kind;
    int ax, ay, bx, by, cx, cy, r;
};

struct shape obj[MAX];
int total = 0;

void init_canvas() {
    int i, j;
    for(i = 0; i < HEIGHT; i++)
        for(j = 0; j < WIDTH; j++)
            canvas[i][j] = '_';
}

void put_pixel(int x, int y) {
    if(x >= 0 && x < WIDTH && y >= 0 && y < HEIGHT)
        canvas[y][x] = '*';
}

void draw_line(int x1, int y1, int x2, int y2) {
    int dx, dy, sx, sy, err, e2;
    dx = abs(x2 - x1);
    dy = abs(y2 - y1);
    if(x1 < x2) sx = 1; else sx = -1;
    if(y1 < y2) sy = 1; else sy = -1;
    err = dx - dy;

    while(1) {
        put_pixel(x1, y1);
        if(x1 == x2 && y1 == y2) break;
        e2 = 2 * err;
        if(e2 > -dy) { err -= dy; x1 += sx; }
        if(e2 < dx)  { err += dx; y1 += sy; }
    }
}

void draw_rect(int x1, int y1, int x2, int y2) {
    draw_line(x1, y1, x2, y1);
    draw_line(x1, y2, x2, y2);
    draw_line(x1, y1, x1, y2);
    draw_line(x2, y1, x2, y2);
}

void draw_circle(int cx, int cy, int r) {
    int x = 0;
    int y = r;
    int d = 1 - r;

    while(x <= y) {
        put_pixel(cx+x, cy+y);
        put_pixel(cx-x, cy+y);
        put_pixel(cx+x, cy-y);
        put_pixel(cx-x, cy-y);
        put_pixel(cx+y, cy+x);
        put_pixel(cx-y, cy+x);
        put_pixel(cx+y, cy-x);
        put_pixel(cx-y, cy-x);

        if(d < 0)
            d += 2*x + 3;
        else {
            d += 2*(x-y) + 5;
            y--;
        }
        x++;
    }
}

void draw_triangle(int x1, int y1, int x2, int y2, int x3, int y3) {
    draw_line(x1, y1, x2, y2);
    draw_line(x2, y2, x3, y3);
    draw_line(x3, y3, x1, y1);
}

void render() {
    int i;
    init_canvas();
    for(i = 0; i < total; i++) {
        if(obj[i].kind == 1)
            draw_line(obj[i].ax, obj[i].ay, obj[i].bx, obj[i].by);
        else if(obj[i].kind == 2)
            draw_rect(obj[i].ax, obj[i].ay, obj[i].bx, obj[i].by);
        else if(obj[i].kind == 3)
            draw_circle(obj[i].ax, obj[i].ay, obj[i].r);
        else if(obj[i].kind == 4)
            draw_triangle(obj[i].ax, obj[i].ay, obj[i].bx, obj[i].by, obj[i].cx, obj[i].cy);
    }
}

void show_canvas() {
    int i, j;
    for(i = 0; i < HEIGHT; i++) {
        for(j = 0; j < WIDTH; j++)
            putchar(canvas[i][j]);
        putchar('\n');
    }
}

void print_header() {
    printf("2D Graphics Editor\n");
    printf("Canvas size: %d x %d\n", WIDTH, HEIGHT);
}

void print_menu() {
    printf("1. Add object\n");
    printf("2. Delete object\n");
    printf("3. Modify object\n");
    printf("4. Display picture\n");
    printf("5. List objects\n");
    printf("0. Exit\n");
    printf("Enter choice: ");
}

void add_object() {
    int t;
    struct shape s;

    printf("\n");
    printf("Choose shape type:\n");
    printf("1. Line\n");
    printf("2. Rectangle\n");
    printf("3. Circle\n");
    printf("4. Triangle\n");
    printf("Enter shape type: ");
    scanf(" %d", &t);

    s.kind = t;
    s.ax = s.ay = s.bx = s.by = s.cx = s.cy = s.r = 0;

    if(t == 1) {
        printf("Enter x1 y1 x2 y2: ");
        scanf(" %d %d %d %d", &s.ax, &s.ay, &s.bx, &s.by);
    } else if(t == 2) {
        printf("Enter top-left x y and bottom-right x y: ");
        scanf(" %d %d %d %d", &s.ax, &s.ay, &s.bx, &s.by);
    } else if(t == 3) {
        printf("Enter center x y and radius: ");
        scanf(" %d %d %d", &s.ax, &s.ay, &s.r);
    } else if(t == 4) {
        printf("Enter x1 y1 x2 y2 x3 y3: ");
        scanf(" %d %d %d %d %d %d", &s.ax, &s.ay, &s.bx, &s.by, &s.cx, &s.cy);
    } else {
        printf("Invalid shape type.\n");
        return;
    }

    obj[total] = s;
    printf("Object added with index %d.\n", total);
    total++;

    printf("\n");
    print_header();
}

void delete_object() {
    int idx, i;
    printf("Enter index to delete: ");
    scanf(" %d", &idx);

    if(idx < 0 || idx >= total) {
        printf("Invalid index.\n");
    } else {
        for(i = idx; i < total - 1; i++)
            obj[i] = obj[i+1];
        total--;
        printf("Object deleted.\n");
    }

    printf("\n");
    print_header();
}

void modify_object() {
    int idx, t;
    struct shape s;

    printf("Enter index to modify: ");
    scanf(" %d", &idx);

    if(idx < 0 || idx >= total) {
        printf("Invalid index.\n");
        printf("\n");
        print_header();
        return;
    }

    printf("Choose shape type:\n");
    printf("1. Line\n");
    printf("2. Rectangle\n");
    printf("3. Circle\n");
    printf("4. Triangle\n");
    printf("Enter shape type: ");
    scanf(" %d", &t);

    s.kind = t;
    s.ax = s.ay = s.bx = s.by = s.cx = s.cy = s.r = 0;

    if(t == 1) {
        printf("Enter x1 y1 x2 y2: ");
        scanf(" %d %d %d %d", &s.ax, &s.ay, &s.bx, &s.by);
    } else if(t == 2) {
        printf("Enter top-left x y and bottom-right x y: ");
        scanf(" %d %d %d %d", &s.ax, &s.ay, &s.bx, &s.by);
    } else if(t == 3) {
        printf("Enter center x y and radius: ");
        scanf(" %d %d %d", &s.ax, &s.ay, &s.r);
    } else if(t == 4) {
        printf("Enter x1 y1 x2 y2 x3 y3: ");
        scanf(" %d %d %d %d %d %d", &s.ax, &s.ay, &s.bx, &s.by, &s.cx, &s.cy);
    } else {
        printf("Invalid shape type.\n");
        printf("\n");
        print_header();
        return;
    }

    obj[idx] = s;
    printf("Object modified.\n");
    printf("\n");
    print_header();
}

void list_objects() {
    int i;
    for(i = 0; i < total; i++) {
        printf("Index %d: ", i);
        if(obj[i].kind == 1)
            printf("Line (%d,%d) to (%d,%d)\n", obj[i].ax, obj[i].ay, obj[i].bx, obj[i].by);
        else if(obj[i].kind == 2)
            printf("Rectangle (%d,%d) to (%d,%d)\n", obj[i].ax, obj[i].ay, obj[i].bx, obj[i].by);
        else if(obj[i].kind == 3)
            printf("Circle center (%d,%d) radius %d\n", obj[i].ax, obj[i].ay, obj[i].r);
        else if(obj[i].kind == 4)
            printf("Triangle (%d,%d) (%d,%d) (%d,%d)\n", obj[i].ax, obj[i].ay, obj[i].bx, obj[i].by, obj[i].cx, obj[i].cy);
    }
    printf("\n");
    print_header();
}

int main() {
    int ch;

    print_header();

    while(1) {
        print_menu();

        if(scanf(" %d", &ch) != 1) break;

        if(ch == 0) {
            printf("Goodbye.\n");
            break;
        } else if(ch == 1) {
            add_object();
        } else if(ch == 2) {
            delete_object();
        } else if(ch == 3) {
            modify_object();
        } else if(ch == 4) {
            printf("\n");
            render();
            show_canvas();
            printf("\n");
            print_header();
        } else if(ch == 5) {
            list_objects();
        } else {
            printf("Invalid choice.\n");
            printf("\n");
            print_header();
        }
    }

    return 0;
}
