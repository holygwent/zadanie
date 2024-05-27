# Rozwiązanie zadania
Treść code review zawarta w komentarzach:<br />
[HttpPost("delete/{id}")] // zamienił bym na [HttpDelete("/{id}")] pod jedną sciężką można używac różnych HTTP metod pozwala to na krótszą przejrzystą scieżkę np book/id zamiast book/delete/id<br />
public void Delete(uint id) // błąd powino być ActionResult przy komunikacji z bazą danych ważna jest asynchroniczność w kodzie dlatego osobiście używam async Task<ActionResult><br />
{<br />
  User user = _context.Users.FirstOrDefault(user=> user.Id == id); // jako Id używałbym guid<br />
  _context.Users.Remove(user); //brak sprawdzenia  user != null i zwrócenie informacji Not Found 404 jeśli brak usera w bazie najcześciej robie to przez dodanie middleware który łapie moje wyjątki i zwraca status<br />
  _context.SaveChanges();<br />
  Debug.WriteLine($"..."); // takie informacje zapisywałbym w logach tesktowych   _logger.LogInformation("....");<br />
  return OK()<br />
}<br />
Rozwiązanie <br />
[HttpDelete("/{id}")]<br />
public async Task<ActionResult> (uint id)<br />
{<br />
  User user = _context.Users.FirstOrDefault(user=> user.Id == id);<br />
  if(user == null)<br />
    return HttpNotFound();//najlepiej wyrzuć error złapany w middleware gdzie jest logger<br />
  _context.Users.Remove(user);<br />
  _context.SaveChangesAsync();<br />
  _logger.LogInformation("....");<br />
  return OK();<br />
} </br>
<br />
//jak poprawnie sie robi :
//  używa się funkcji z repository pattern servisu 
//logike powinno się przenieść do niższej warstwy np użyć generycznego repository pattern który sprawia że nie trzeba dla każdej klasy osobno później pisać tego samego kodu dla metod CRUD<br />
 public class GenericRepository<T> : IGenericRepository<T> where T : class <br />
    {<br />
        private readonly ApplicationDbContext _context;<br />
        private readonly DbSet<T> _entities;<br />
        private readonly ILogger<GenericRepository<T>> _logger;<br />
<br />
        public GenericRepository(ApplicationDbContext context, ILogger<GenericRepository<T>> logger)<br />
        {<br />
            _context = context;<br />
            _entities = _context.Set<T>();<br />
            _logger = logger;<br />
        }<br />
<br />
        public void Delete(object id)<br />
        {<br />
<br />
            var entity = GetByIdAsync(id).Result;<br />
            if (entity == null)<br />
                throw new NotFoundException($"there is no entity with id {id}");<br />
<br />
            _logger.LogWarning($"Entity with id:{id} DELETE action invoke");<br />
            EntityEntry entityEntry = _context.Entry<T>(entity);<br />
            entityEntry.State = EntityState.Deleted;<br />
            SaveChangesAsync().Wait();<br />
<br />
        }<br />
}<br />
