# Лаборатори №5: API систем тест (Postman ба Newman)

- Нэр: Д. Баттөмөр
- Код: b242270025

```
$ node -v
v20.19.2
$ newman -v
6.2.3
```

## Тест дизайн

### Сонголт ба төлөөлөх утгын хүснэгт

| Сонголт | Эквивалент анги | Төлөөлөх утга |
| --- | --- | --- |
| studentID-гийн хүчинтэй байдал | идэвхтэй | B242270025, status="active" |
| | идэвхгүй | B242270025, status="inactive" |
| | байхгүй | UNKNOWN999 |
| Оюутны үзсэн хичээл | урьдач хангана (урьдачгүй хичээл) | coursesTaken=[] , prerequisites=[] |
| | урьдач хангахгүй | coursesTaken=[""], prerequisites=["CS201"] |
| courseID-гийн хүчинтэй байдал | байгаа | CS313, C_missing, C_noprereq |
| | байхгүй | UNKNOWN999 |
| Хичээлийн урьдач нөхцөл | огт үзээгүй | S_missing: prerequisites=["CS201"], coursesTaken үгүй |
| | урьдачгүй | prerequisites=[] |
| Хүсэлтийн формат | studentID талбар дутуу | зөвхөн {"courseID":"CS313"} |

### Спецификацийн хүснэгт

| # | Спецификаци (тестийн нэр) | Оролт | Хүлээгдэх статус | Хүлээгдэх result |
| --- | --- | --- | --- | --- |
| 1 | Happy path (Test - Successful Registration) | идэвхтэй оюутан, байгаа хичээл | 201 | OK, registrationID тоо |
| 2 | Оюутан байхгүй (Test - No Student) | studentID=UNKNOWN999 | 200 | ERROR_NO_STUDENT |
| 3 | Идэвхгүй оюутан (Test - Inactive Student) | status="inactive" | 200 | ERROR_INACTIVE_STUDENT |
| 4 | Хичээл байхгүй (No Course) | courseID=UNKNOWN999 | 200 | ERROR_NO_COURSE |
| 5 | Урьдач нөхцөл дутуу (Test - Missing Prerequisite) | S_missing, C_missing (prereq CS201) | 200 | ERROR_PREREQUISITES, missing=["CS201"] |
| 6 | Давхар алдаа: идэвхгүй оюутан + байхгүй хичээл (Test - Combined Error) | inactive, UNKNOWN999 | 200 | ERROR_INACTIVE_STUDENT (оюутны алдаа давамгайлна) |
| 7 | Хязгаар: studentID талбар дутуу (Test - Bad Request) | зөвхөн courseID | 400 | ERROR_BAD_REQUEST |
| 8 | Хязгаар: урьдачгүй хичээл, хоосон coursesTaken (Test - No Prerequisites) | S_noprereq, C_noprereq | 201 | OK |
