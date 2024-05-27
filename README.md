# Rozwiązanie zadania
Treść code review zawarta w komentarzach:<br />
[HttpPost("delete/{id}")] // lepiej [HttpDelete("/{id}")] pod jedną sciężką można używac różnych HTTP metod 
public void Delete(uint id) // niepoprawny zwracany typ powinien być ActionTask a lepiej asynchroniczny, przy komunikacji z bazą danych ważna jest asynchroniczność<br />
{//brakuje w parametrach atrybutu<br />
  User user = _context.Users.FirstOrDefault(user=> user.Id == id); // brak tutaj funkcji asynchronicznej<br />
  _context.Users.Remove(user); //brak sprawdzenia  user != null i zwrócenie  Not Found 404 
  _context.SaveChanges();<br />//brak asynchronicznej funkcji
  Debug.WriteLine($"..."); // takie informacje zapisywałbym w logach tesktowych
  return OK()<br />
}<br />
Rozwiązanie <br />
[HttpDelete("/{id}")]<br />
public async Task<ActionResult> ([FromRoute]uint id)<br />
{<br />
  User user = await _context.Users.FirstOrDefaultAsync(user=> user.Id == id);<br />
  if(user == null)<br />
    return HttpNotFound();//najlepiej wyrzucić error złapany w middleware gdzie jest logger<br />
  _context.Users.Remove(user);<br />
  _context.SaveChangesAsync();<br />
  _logger.LogInformation("....");<br />
  return OK();<br />
} </br>
<br />

