# Rozwiązanie zadania
Treść code review zawarta w komentarzach:
//logike powinno się przenieść do niższej warstwy np użyć generycznego repository pattern który sprawia że nie trzeba dla każdej klasy osobno później pisać tego samego kodu dla metod CRUD
[HttpPost("delete/{id}")] // zamienił bym na [HttpDelete("/{id}")] pod jedną sciężką można używac różnych HTTP metod pozwala to na krótszą przejrzystą scieżkę np book/id zamiast book/delete/id
public void Delete(uint id) // błąd powino być ActionResult przy komunikacji z bazą danych ważna jest asynchroniczność w kodzie dlatego osobiście używam async Task<ActionResult>
{
  User user = _context.Users.FirstOrDefault(user=> user.Id == id); // jako Id używałbym guid
  _context.Users.Remove(user); //brak sprawdzenia  user != null i zwrócenie informacji Not Found 404 jeśli brak usera w bazie najcześciej robie to przez dodanie middleware który łapie moje wyjątki i zwraca status
  _context.SaveChanges();
  Debug.WriteLine($"..."); // takie informacje zapisywałbym w logach tesktowych   _logger.LogInformation("....");
  return OK()
}
Rozwiązanie 
[HttpDelete("/{id}")]
public async Task<ActionResult> (uint id)
{
  User user = _context.Users.FirstOrDefault(user=> user.Id == id);
  if(user == null)
    return HttpNotFound();//najlepiej wyrzuć error złapany w middleware gdzie jest logger
  _context.Users.Remove(user);
  _context.SaveChangesAsync();
  _logger.LogInformation("....");
  return OK();
}
