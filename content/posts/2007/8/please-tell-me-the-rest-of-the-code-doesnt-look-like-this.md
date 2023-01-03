+++
title = "Please tell me the rest of the code doesn't look like this..."
date = 2007-08-10T13:47:00.003-05:00
updated = 2008-04-10T18:49:40.712-05:00
draft = false
url = '/2007/08/please-tell-me-rest-of-code-doesn-look.html'
tags = ["SQL","Microsoft","injection","bug","best practice"]
+++

This is **not** right:

```
internal static bool DoesDbExist(SqlConnection conn, string database){    using (SqlCommand cmd = conn.CreateCommand())    {        // prefer this to a where clause as this is not prone to injection attacks        cmd.CommandText = "SELECT name FROM sys.databases";        cmd.CommandType = CommandType.Text; 
        using (SqlDataReader reader = cmd.ExecuteReader())        {            while (reader.Read())            {                string dbName = reader.GetString(0);                if (string.Compare(dbName, database, true, CultureInfo.CurrentCulture) == 0)                {                    // the database already exists - return                    return true;                }            }        }    } 
    return false;}
```

This **is** right:

```
internal static bool DoesDbExist(SqlConnection conn, string database){    using (SqlCommand cmd = conn.CreateCommand())    {        cmd.CommandText = "SELECT name FROM sys.databases WHERE name=@name";        cmd.CommandType = CommandType.Text;        cmd.Parameters.Add(new SqlParameter("@name", database)); 
        using (SqlDataReader reader = cmd.ExecuteReader())        {            return reader.Read();        }    }}
```

Someone please assure me that this is not how everyone else handles avoiding SQL injection.

---
### Comments:
#### That's pretty awesome. (Aren't SqlParameters great?)
[Brian Dukes](https://www.blogger.com/profile/07026014589345008284 "noreply@blogger.com") - <time datetime="2007-08-15T09:00:00.000-05:00">Aug 3, 2007</time>

That's pretty awesome. (Aren't SqlParameters great?)
<hr />
#### Hahaha. I was reading your blog. This is great. ...
[The Software Purist (http://www.softwarepurist.com)]( "noreply@blogger.com") - <time datetime="2009-12-02T07:14:54.000-06:00">Dec 3, 2009</time>

Hahaha. I was reading your blog. This is great. I've seen things like this done before and it always makes me want to cry. :P In the particular case I'm thinking of, it was in C++, but the concept is the same. In my own experience, I typically use stored procedures, which mitigates a lot of the risk you were mentioning.
<hr />
