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

## Newman үр дүн

| Ажиллуулалт | Файл | Collection | assertions executed | assertions failed | exit code |
| --- | --- | --- | --- | --- | --- |
| PASS | results/newman-pass.txt | lab05-collection.json | 17 | 0 | 0 |
| FAIL | results/newman-fail.txt | lab05-collection-fail.json | 17 | 1 | 1 |
| DOWN | results/newman-down.txt | lab05-collection.json | 17 | 17 | 1 |

Collection-д нийт 8 тест, 17 assertion байна (PASS гаралтын assertions executed тоотой таарна).
FAIL ажиллуулалтад `Status code is 201` шалгалтыг зориуд 200 болгосон тул 1 assertion унаж, exit code 1 болсон. Энэ нь CI quality gate-ийн зарчим: нэг oracle унавал pipeline зогсоно.
DOWN ажиллуулалтад сервер унтарсан тул гаралтад ECONNREFUSED гарсан. Энэ нь интерфейсийн алдаа (серверт холбогдож чадаагүй), oracle-ийн алдаа биш (хариу ирж, хүлээгдэх утгатай таарахгүй байсан зүйл биш).

## Дүгнэлт

Дизайны 5 алхмаас хамгийн их бодол шаардсан нь сонголтуудыг эквивалент ангид хуваах алхам байлаа, учир нь аль хослол бодит тест болохыг энд шийддэг. Боломжгүй хослол таарсан: байхгүй оюутанд урьдач нөхцөл шалгах боломжгүй, учир нь оюутны бичлэггүй бол coursesTaken байхгүй. Давхар алдааны хослолд (идэвхгүй оюутан + байхгүй хичээл) сервер ERROR_INACTIVE_STUDENT буцааж, оюутны шалгалт хичээлийн шалгалтаас өмнө хийгддэгийг тестээр тогтоосон. Тестүүд server.js-д согог олсонгүй, бүх 17 assertion дамжсан. Харин өөрийн тестийн дизайнд алдаа илрүүлсэн: Setup - Missing Prerequisite Course нь POST-ийн дараа байсан тул шинэ сервер дээр ERROR_NO_COURSE буцаж нэг assertion унасан. Өмнөх ажиллуулалтын өгөгдөл серверийн санах ойд үлдсэн байсан тул серверийг дахин асаах хүртэл энэ нь мэдэгдээгүй. Үүнийг setup-уудыг тест бүрийн өмнө байрлуулж засаж, тестүүдийг бие даасан болгосон. Мөн collection-ы baseUrl хувьсагч хоосон утгатай export болсон тул Newman анх Invalid URI алдаа өгсөн бөгөөд утгыг нь засаж шийдсэн. FAIL ажиллуулалт нэг oracle буруу байхад exit code 1 болдгийг, DOWN ажиллуулалт сервер унтарсан үед ECONNREFUSED гардгийг харуулсан. Эндээс автомат тест нь CI-д quality gate болж чаддагийг ойлгосон.
