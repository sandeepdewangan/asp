# Relationships

## Models
```csharp
public class Book
    {
        public int BookId { get; set; }
        public string Title { get; set; }
        public string ISBN { get; set; }
        public double Price { get; set; }

        public int BookDetailId { get; set; }
        public int PublisherId { get; set; }


        // Navigation prop
        public BookDetail BookDetail { get; set; }
        public Publisher Publisher { get; set; }

        // for many to many
        public virtual ICollection<BookAuthor> BookAuthor { get; set; }

    }

public class BookDetail
    {
        public int BookDetailId { get; set; }
        public int NumberOfChapters { get; set; }
        public int NumberOfPages { get; set; }
        public double Weight { get; set; }

        // Navigation prop
        public Book Book { get; set; }
    }
    
    public class Author
    {
        public int AuthorId { get; set; }
        public string Name { get; set; }
        [NotMapped]
        public string FullName { get; set; }

        // for many to many
        public virtual ICollection<BookAuthor> BookAuthor { get; set; }

    }
    
    public class Publisher
    {
        public int PublisherId { get; set; }
        public string Name { get; set; }


        public List<Book> Book { get; set; }
    }
    
    public class BookAuthor
    {
        public int BookId { get; set; }
        public int AuthorId { get; set; }

        // Navigation pro
        public Book Book { get; set; }
        public Author Author { get; set; }
    }
```
## Db Context - fluent API
```csharp
protected override void OnModelCreating(ModelBuilder builder)
        {
            base.OnModelCreating(builder);


            // BookDetail Table
            // setting BookDetailId as primary key.
            builder.Entity<BookDetail>().HasKey(b => b.BookDetailId);
            // setting NumberOfPages column as required field.
            builder.Entity<BookDetail>().Property(b => b.NumberOfPages).IsRequired();

            // Book Table
            builder.Entity<Book.Book>().HasKey(b => b.BookId);
            builder.Entity<Book.Book>().Property(b => b.ISBN).IsRequired().HasMaxLength(15);

            // One to one relation between Book and Book Detail
            builder.Entity<Book.Book>().HasKey(b => b.BookId);
            builder.Entity<Book.Book>().HasOne(b => b.BookDetail).WithOne(c => c.Book).HasForeignKey<Book.Book>(f => f.BookDetailId);

            // One to many relationship between publisher and book.
            // A publisher can have multiple books but a book can have only one publisher.
            builder.Entity<Book.Book>().HasOne(z => z.Publisher).WithMany(z => z.Book).HasForeignKey(z => z.PublisherId);

            // Many to Many Relationship between Book and Author, Intermediate table BookAuthor.
            builder.Entity<BookAuthor>().HasKey(b => new { b.BookId, b.AuthorId });
            builder.Entity<BookAuthor>().HasOne(z => z.Book).WithMany(z => z.BookAuthor).HasForeignKey(z => z.BookId);
            builder.Entity<BookAuthor>().HasOne(z => z.Author).WithMany(z => z.BookAuthor).HasForeignKey(z => z.AuthorId);
        }
```



