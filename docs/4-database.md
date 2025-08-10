# Database & ERD

## Tabel Utama
- users (id, name, email, role, password)
- students (id, user_id, nim, program_studi_id, angkatan, status)
- lecturers (id, user_id, nidn, fakultas_id, status)
- faculties (id, name)
- study_programs (id, name, faculty_id)
- courses (id, code, name, sks, semester, prerequisites)
- schedules (id, course_id, lecturer_id, day, start_time, end_time, room)
- krs (id, student_id, semester, academic_year)
- krs_items (id, krs_id, schedule_id)
- khs (id, student_id, semester, academic_year, gpa, total_sks)
- grades (id, khs_id, course_id, grade, weight)
- payments (id, student_id, amount, method, date, description)

## Relasi
- 1 mahasiswa → banyak KRS & KHS
- 1 dosen → banyak jadwal mengajar
- 1 program studi → banyak mahasiswa
