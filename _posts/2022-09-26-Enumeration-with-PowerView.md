## Enumeration Using PowerView

This blog post includes post exploitation enumeration methods using PowerView that could help in lateral movement and data hunting.

---

### Getting Started

If you try running PowerView script with default settings and configuration, you might encounter error, one such example is below.

![Error ng](https://user-images.githubusercontent.com/60311143/192285693-55bb4e49-1302-465f-b691-f30044e162f1.PNG)

We can get pass this error by changing the execution policy to `RemoteSigned`. This allows running scripts.
```
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```
![1](https://user-images.githubusercontent.com/60311143/192286179-d4a4730b-7d4d-40be-9695-315e2a2054f6.png)

### Get that Domain!!!

To get a domain object of the present domain run the following. Get details of a partiular domain by running the command with `-Domain'.
```
Get-Domain
```
![2](https://user-images.githubusercontent.com/60311143/192290135-3fc70be5-ecba-4167-a17a-056cfc98c273.png)
