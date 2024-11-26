# mynote
this repo contains all about programming notes

---

# installing PostgreSQL

Create the file repository configuration:

`sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'`

Import the repository signing key:

`wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -`

Update the package lists:

`sudo apt-get update`

Install the latest version of PostgreSQL.
If you want a specific version, use 'postgresql-12' or similar instead of 'postgresql':

`sudo apt-get -y install postgresql`

---

# PostgreSQL error
> FATAL:  role "reqishere" does not exist
> Couldn't create 'MyBlog_development' database.

login as su

`sudo su - postgres`

run psql

`psql -U postgres`

when you got this postgres=#, run this command:

`create role reqishere with createdb login password 'yourpassword';`

now, you can test it.

---

    
4. then run this to update grub:
   `sudo update-grub`
   
5. save then reboot the system
   `reboot`
