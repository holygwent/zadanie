# Rozwiązanie zadania
Treść code review zawarta w komentarzach:<br />
[HttpPost("delete/{id}")] // lepiej [HttpDelete("/{id}")] pod jedną sciężką można używac różnych HTTP metod<br/>
public void Delete(uint id) //niepoprawny typ, ma być ActionTask,lepiej asynchroniczny przy komunikacji z bazą<br />
{//brakuje w parametrach atrybutu<br />
  User user = _context.Users.FirstOrDefault(user=> user.Id == id); // brak tutaj funkcji asynchronicznej<br />
  _context.Users.Remove(user); //brak sprawdzenia  user != null i zwrócenie  Not Found 404 lub error<br/>
  _context.SaveChanges();//brak asynchronicznej funkcji<br/>
  Debug.WriteLine($"..."); // takie informacje zapisywałbym w logach tesktowych<br/>
  return OK()<br />
}<br /><br />
Rozwiązanie: <br />
[HttpDelete("/{id}")]<br />
public async Task&lt;ActionResult&gt; ([FromRoute]uint id)<br />
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

