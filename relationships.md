# Relationships

### One to One Relationship
```csharp
public class Book
    {
        public int BookId { get; set; }
        public string Title { get; set; }

        [ForeignKey("BookDetail")]
        public int BookDetailId { get; set; }
        public BookDetail BookDetail { get; set; }
    }
}
```
```csharp
public class BookDetail
    {
        [Key]
        public int Id { get; set; }

        // navigation property
        public Book Book { get; set; }
    }
```

### One to Many Relationship
```csharp
public class Book
    {
        public int BookId { get; set; }

        // Book shall have one publisher
        [ForeignKey("BookPublisher")]
        public int PublisherId { get; set; }
        public BookPublisher BookPublisher { get; set; }
    }
```
```csharp
public class BookPublisher
    {
        [Key]
        public int Id { get; set; }
        public string Name { get; set; }

        // Publisher can publish many books
        public List<Book> Books { get; set; }
    }
```

