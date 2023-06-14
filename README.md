# Demo_code_RazorPage

## Bước 1: Chuẩn bị cơ sở dữ liệu

Trước tiên, bạn cần tạo cơ sở dữ liệu SQL Server và tạo các bảng Students (sinh viên), Courses (khóa học) và bảng trung gian StudentCourses (đăng ký khóa học của sinh viên) với các trường tương ứng. Bảng StudentCourses sẽ có hai trường khóa ngoại là StudentId và CourseId, liên kết với bảng Students và Courses tương ứng.

## Bước 2: Tạo một dự án Razor Pages mới

Lưu ý add đủ thư viện

```C#
Microsoft.EntityFrameworkCore
```

```C#
Microsoft.EntityFrameworkCore.SqlServer
```

```C#
Microsoft.EntityFrameworkCore.Tools
```

## Bước 3: Tạo model và DbContext

```c#
public class Student
{
    public int StudentId { get; set; }
    public string Name { get; set; }
    public ICollection<StudentCourse> StudentCourses { get; set; }
}
```

```c#
public class Student
    public class Course
    {
        public int CourseId { get; set; }
        public string Title { get; set; }
        public ICollection<StudentCourse> StudentCourses { get; set; }
    }
```

```c#
public class Student
    public class StudentCourse
    {
        public int StudentId { get; set; }
        public Student Student { get; set; }

        public int CourseId { get; set; }
        public Course Course { get; set; }
    }
```

```c#
public class Student
    public class StudentCourse
    {
        public int StudentId { get; set; }
        public Student Student { get; set; }

        public int CourseId { get; set; }
        public Course Course { get; set; }
    }
```

```c#
 public static class AppDbInitializer
    {
        public static void SeedData(this ModelBuilder builder)
        {
            builder.Entity<Course>().HasData(
                new Course
                {
                    CourseId = 1,
                    Title = "PRO"
                },
                new Course
                {
                    CourseId = 2,
                    Title = "PRO"
                },
                new Course
                {
                    CourseId = 3,
                    Title = "PRO"
                }
                );
            builder.Entity<Student>().HasData(
               new Student
               {
                   StudentId = 1,
                   Name = "Nguyen A"
               },
               new Student
               {
                   StudentId = 2,
                   Name = "Nguyen B"
               },
               new Student
               {
                   StudentId = 3,
                   Name = "Nguyen C"
               }
               );
            builder.Entity<StudentCourse>().HasData(
              new StudentCourse
              {
                  StudentId = 1,
                  CourseId = 1
              },
              new StudentCourse
              {
                  StudentId = 1,
                  CourseId = 2
              },
              new StudentCourse
              {
                  StudentId = 2,
                  CourseId = 2
              },
              new StudentCourse
              {
                  StudentId = 2,
                  CourseId = 3
              },
              new StudentCourse
              {
                  StudentId = 3,
                  CourseId = 3
              }
              );
        }
    }
```

```c#
 public class AppDbContext : DbContext
    {
        public AppDbContext()
        {
        }

        public AppDbContext(DbContextOptions options) : base(options)
        {

        }

        /// <summary>
        /// định nghĩa Course trong cơ sở dữ liệu.
        /// </summary>
        public DbSet<Course> Courses { get; set; }
        public DbSet<Student> Students { get; set; }
        public DbSet<StudentCourse> StudentCourses { get; set; }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            base.OnConfiguring(optionsBuilder);
            if (optionsBuilder.IsConfigured)
            {
                optionsBuilder.UseSqlServer("server=.;database=DemoPRN;uid=sa;pwd=123;MultipleActiveResultSets=True");
            }
        }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);

            // Cấu hình cho lớp StudentCourse.
            modelBuilder.Entity<StudentCourse>()
                .HasKey(sc => new { sc.StudentId, sc.CourseId });

            modelBuilder.Entity<StudentCourse>()
                .HasOne(sc => sc.Student)
                .WithMany(s => s.StudentCourses)
                .HasForeignKey(sc => sc.StudentId);

            modelBuilder.Entity<StudentCourse>()
                .HasOne(sc => sc.Course)
                .WithMany(c => c.StudentCourses)
                .HasForeignKey(sc => sc.CourseId);
            modelBuilder.SeedData();
        }
    }
```

```c#
var builder = WebApplication.CreateBuilder(args);

var connectionString = builder.Configuration.GetConnectionString("MyCnn");
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));

// Add services to the container.
builder.Services.AddRazorPages();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

app.UseRouting();

app.UseAuthorization();

app.MapRazorPages();

app.Run();
```

- Tạo Migration `add-migration` + với tên Database ( nó không liên quan gì đến việc đặt tên database )

```C#
add-migration DBJustBlog
```

- Cập nhập xuống database

```C#
update-database
```
