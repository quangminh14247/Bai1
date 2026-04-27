#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 1. Khai báo cấu trúc dữ liệu
typedef struct Ngay {
    int ngay, thang, nam;
} Ngay;

typedef struct SinhVien {
    char maSV[8];
    char hoTen[50];
    int gioiTinh; // 0: Nam, 1: Nu
    Ngay ngaySinh;
    char diaChi[100];
    char lop[12];
    char khoa[7];
} SinhVien;

typedef struct Node {
    SinhVien data;
    struct Node *link;
} Node;

typedef struct List {
    Node *first;
    Node *last;
} List;

// Khởi tạo danh sách rỗng
void init(List *l) {
    l->first = l->last = NULL;
}

// Tạo node mới
Node* createNode(SinhVien sv) {
    Node *p = (Node*)malloc(sizeof(Node));
    if (p == NULL) return NULL;
    p->data = sv;
    p->link = NULL;
    return p;
}

// Hàm nhập thông tin 1 sinh viên
SinhVien nhapSV() {
    SinhVien sv;
    printf("Ma SV: "); scanf("%s", sv.maSV);
    getchar(); // xoa bo nho dem
    printf("Ho ten: "); fgets(sv.hoTen, sizeof(sv.hoTen), stdin);
    sv.hoTen[strcspn(sv.hoTen, "\n")] = 0;
    printf("Gioi tinh (0-Nam, 1-Nu): "); scanf("%d", &sv.gioiTinh);
    printf("Ngay thang nam sinh: "); scanf("%d %d %d", &sv.ngaySinh.ngay, &sv.ngaySinh.thang, &sv.ngaySinh.nam);
    getchar();
    printf("Dia chi: "); fgets(sv.diaChi, sizeof(sv.diaChi), stdin);
    sv.diaChi[strcspn(sv.diaChi, "\n")] = 0;
    printf("Lop: "); scanf("%s", sv.lop);
    printf("Khoa: "); scanf("%s", sv.khoa);
    return sv;
}

// 2. Them sinh vien vao danh sach da sap xep tang dan theo maSV
void insertSorted(List *l, SinhVien sv) {
    Node *p = createNode(sv);
    if (l->first == NULL) {
        l->first = l->last = p;
        return;
    }
    
    // Them vao dau neu maSV nho nhat
    if (strcmp(p->data.maSV, l->first->data.maSV) < 0) {
        p->link = l->first;
        l->first = p;
        return;
    }

    // Tim vi tri thich hop
    Node *curr = l->first;
    Node *prev = NULL;
    while (curr != NULL && strcmp(curr->data.maSV, p->data.maSV) < 0) {
        prev = curr;
        curr = curr->link;
    }
    
    prev->link = p;
    p->link = curr;
    if (curr == NULL) l->last = p;
}

// Ham in thong tin sinh vien
void inSV(SinhVien sv) {
    printf("%-10s | %-20s | %d/%d/%-5d | %s\n", sv.maSV, sv.hoTen, sv.ngaySinh.ngay, sv.ngaySinh.thang, sv.ngaySinh.nam, sv.lop);
}

// 3. In cac sinh vien co cung ngay sinh
void inSVCoCungNgaySinh(List l) {
    int found = 0;
    for (Node *p = l.first; p != NULL; p = p->link) {
        int count = 0;
        for (Node *q = l.first; q != NULL; q = q->link) {
            if (p != q && p->data.ngaySinh.ngay == q->data.ngaySinh.ngay &&
                p->data.ngaySinh.thang == q->data.ngaySinh.thang &&
                p->data.ngaySinh.nam == q->data.ngaySinh.nam) {
                count++;
            }
        }
        if (count > 0) {
            inSV(p->data);
            found = 1;
        }
    }
    if (!found) printf("Khong tim thay sinh vien cung ngay sinh\n");
}

// 4. Loai bo sinh vien cung ngay sinh
void xoaSVTrungNgaySinh(List *l) {
    Node *curr = l->first;
    Node *prev = NULL;

    while (curr != NULL) {
        int hasDuplicate = 0;
        for (Node *q = l->first; q != NULL; q = q->link) {
            if (curr != q && curr->data.ngaySinh.ngay == q->data.ngaySinh.ngay &&
                curr->data.ngaySinh.thang == q->data.ngaySinh.thang &&
                curr->data.ngaySinh.nam == q->data.ngaySinh.nam) {
                hasDuplicate = 1;
                break;
            }
        }

        if (hasDuplicate) {
            Node *temp = curr;
            if (prev == NULL) {
                l->first = curr->link;
                curr = l->first;
            } else {
                prev->link = curr->link;
                curr = prev->link;
            }
            if (temp == l->last) l->last = prev;
            free(temp);
        } else {
            prev = curr;
            curr = curr->link;
        }
    }
}

int main() {
    List listSV;
    init(&listSV);
    int n;

    printf("Nhap so luong sinh vien: ");
    scanf("%d", &n);

    for (int i = 0; i < n; i++) {
        printf("\nNhap sinh vien thu %d:\n", i + 1);
        insertSorted(&listSV, nhapSV());
    }

    printf("\nDanh sach sinh vien sau khi sap xep:\n");
    for (Node *p = listSV.first; p != NULL; p = p->link) inSV(p->data);

    printf("\nCac sinh vien co cung ngay sinh:\n");
    inSVCoCungNgaySinh(listSV);

    printf("\nLoai bo sinh vien cung ngay sinh...");
    xoaSVTrungNgaySinh(&listSV);

    printf("\nDanh sach sau khi loai bo:\n");
    for (Node *p = listSV.first; p != NULL; p = p->link) inSV(p->data);

    return 0;
}
