---
layout: post
title: "Getting and inserting MySQL data with perl CGI"
date: 2010-04-13
comments: true
categories: linux other
---
{% img left /images/blog_posts/mysql.png %}

A handy way of capturing data with a browser and inserting into MySQL via form submit. Pulling the captured data from MySQL and displaying it with a browser is just as easy.
<!--more-->
<br>
<br>

##Creating the database and tables:

{% codeblock lang:mysql %}
mysql -u root -p
mysql> CREATE DATABASE tv;
mysql> USE tv;
mysql> CREATE TABLE episodeguide (id INT NOT NULL AUTO_INCREMENT PRIMARY KEY, tvshow VARCHAR(50), shownumber VARCHAR(20), showtitle VARCHAR(50), aired VARCHAR(20), overview VARCHAR(500));
{% endcodeblock %}

**NOTE:** The database is set up to auto increment the row id, with every new row that's inserted.

##Creating Indexes:

{% codeblock lang:mysql %}
mysql> CREATE INDEX tvshow ON episodeguide (tvshow);
mysql> CREATE INDEX showtitle ON episodeguide (showtitle);
mysql> CREATE INDEX overview ON episodeguide (overview);
mysql> exit
{% endcodeblock %}

###Inserting data into the MySQL database with Perl/CGI/HTML:

The HTML file; Call it insert.html and put it in the root of the website.

{% codeblock lang:html %}
<form action="https://192.168.1.1/cgi-bin/insertdata.pl" method="post"> 
<html>    
<head>     
     <title>TV Episode Data Capture Form</title>     
</head>     
<body> 
   <H1>TV Episode Data Capture Form</H1>    
 <P>     
 <HR> 
   <H2>Episode submission form: </H2>    
     TV Show Name:     
       <select name="tvshow">     
        <option selected>SHOW     
        <option>TV Show 1     
        <option>TV Show 2     
        <option>TV Show 3     
       </select>     
 <P> 
     Episode Number:    
       <select name="shownumber">     
        <option selected>NUMBER     
        <option>01     
        <option>02     
        <option>03     
        <option>04     
        <option>05     
        <option>06     
        <option>07     
        <option>08     
        <option>09     
        <option>10     
        <option>11     
        <option>12     
        <option>13     
        <option>14     
        <option>15          
       </select> 
 <P>    
     Episode Title: <input name="showtitle"><P>     
     Original Aired: <input name="aired"><P> 
 <P>    
     Episode Description:     
 <P>     
      <textarea name="overview" rows=5 cols=60></textarea>     
 <P> 
      <input type="submit" value="Click to submit entry"> 
</form>
{% endcodeblock %}

The Perl/CGI file; This must be called insertdata.pl and be placed in the cgi-bin directory of your website.

{% codeblock lang:perl %}
#!/usr/bin/perl    
use CGI qw(:standard);     
use DBI; 

print <<END;    
Content-type: text/html 
<html>    
<head>     
<title>TV Episode capture Form</title>     
</head> 
<body bgcolor="white"> 
END 

# database connection info    
$db="tv";     
$host="localhost";     
$userid="mysql_username";     
$passwd="mysql_password";     
$connectionInfo="dbi:mysql:$db;$host"; 
$tvshow = param('tvshow');    
$shownumber = param('shownumber');     
$showtitle = param('showtitle');     
$aired = param('aired');     
$overview = param('overview');     
$year = param('minutesstart');     
$month = param('yearstop');     
$day = param('monthstop');

# connect to database    
$dbh = DBI->connect($connectionInfo,$userid,$passwd);

# prepare and execute query    
$query = "INSERT INTO episodeguide (tvshow,shownumber,showtitle,aired,overview) VALUES(\"$tvshow\",\"Episode $shownumber\",\"$showtitle\",\"$aired\",\"$overview\");";     
$sth = $dbh->prepare($query) || die "Could not prepare SQL statement ... maybe invalid?";     
$sth->execute() || die "Could not execute SQL statement ... maybe invalid?";

# assign fields to variables    
$sth->bind_columns(\$ID, \$tvshow, \$shownumber, \$showtitle, \$aired, \$overview);

# output thanks message to browser    
print "<b>Your entry was sucessfully captured in the database!</b><p>\n";     
print "</table>\n";     
print "</body>\n";     
print "</html>\n"; 
$sth->finish();

# disconnect from database    
$dbh->disconnect; 
sub fail {    
print "<title>Error</title>",     
"<p>ERROR: An error ocured, please try again!</p>";     
exit; }
{% endcodeblock %}

Giving the Perl script execution permissions:

{% codeblock lang:bash %}
chmod 755 /var/www/cgi-bin/insertdata.pl
{% endcodeblock %}

Go to `http://192.168.1.1/insert.html` to test.

##Displaying the inserted data with Perl/CGI:

{% codeblock lang:bash %}
vi /var/www/cgi-bin/displaydata.pl
{% endcodeblock %}

{% codeblock lang:perl %}
#!/usr/bin/perl -w 
use DBI;

print <<END;    
Content-type: text/html 
<html>    
<head>     
<title>Display my data!</title>     
</head> 
<body bgcolor="white"> 
END

# database connection info    
$db="tv";     
$host="localhost";     
$userid="mysql_username";     
$passwd="mysql_password";     
$connectionInfo="dbi:mysql:$db;$host";

# connect to database    
$dbh = DBI->connect($connectionInfo,$userid,$passwd);

# prepare and execute query    
$query = "SELECT * FROM episodeguide WHERE tvshow = 'Test TV Show'";     
$sth = $dbh->prepare($query);     
$sth->execute();

# assign fields to variables    
$sth->bind_columns(\$ID, \$tvshow, \$shownumber, \$showtitle, \$aired, \$overview);

# output result to browser    
print "<b>Matches for this hardcoded query:</b><p>\n";     
print "<table width=\"100\%\" border=\"1\" cellpadding=\"2\" cellspacing=\"0\"> <th>ID</th> <th>Show Name</th> <th>Episode Number</th> <th>Episode Title</th> <th>Date Aired</th> <th>Episode Overview</th>\n";     
while($sth->fetch()) {     
print "<tr><td>$ID</td> <td>$tvshow</td> <td>$shownumber</td> <td>$showtitle</td> <td>$aired</td> <td>$overview</td> </tr>\n";     
}     
print "</table>\n";     
print "</body>\n";     
print "</html>\n"; 
$sth->finish();

# disconnect from database    
$dbh->disconnect;
{% endcodeblock %}

Assign execution permissions to the Perl script.

{% codeblock lang:bash %}
chmod 755 /var/www/cgi-bin/displaydata.pl
{% endcodeblock %}

And test: `http://192.168.1.1/cgi-bin/displaydata.pl`
