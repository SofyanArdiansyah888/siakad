# API & tRPC Router

## Example Routers

### Student Routes
- student.create(input: StudentInput) → Student
- student.read(id: string) → Student
- student.update(id: string, input: StudentInput) → Student
- student.delete(id: string) → boolean
- student.list(filters) → Student[]

### Lecturer Routes
- lecturer.create(input: LecturerInput) → Lecturer
- lecturer.read(id: string) → Lecturer
- lecturer.update(id: string, input: LecturerInput) → Lecturer
- lecturer.delete(id: string) → boolean
- lecturer.list(filters) → Lecturer[]

### Course Routes
- course.create(input: CourseInput) → Course
- course.read(id: string) → Course
- course.update(id: string, input: CourseInput) → Course
- course.delete(id: string) → boolean
- course.list(filters) → Course[]

### KRS Routes
- krs.create(input: KRSInput) → KRS
- krs.read(id: string) → KRS
- krs.update(id: string, input: KRSInput) → KRS
- krs.delete(id: string) → boolean
- krs.submit(input) → KRS
- krs.list(filters) → KRS[]

### KHS Routes
- khs.create(input: KHSInput) → KHS
- khs.read(id: string) → KHS
- khs.update(id: string, input: KHSInput) → KHS
- khs.delete(id: string) → boolean
- khs.get(student_id, semester) → KHS
- khs.list(filters) → KHS[]

### Payment Routes
- payment.create(input: PaymentInput) → Payment
- payment.read(id: string) → Payment
- payment.update(id: string, input: PaymentInput) → Payment
- payment.delete(id: string) → boolean
- payment.list(filters) → Payment[]

## Error Format
