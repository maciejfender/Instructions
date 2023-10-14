Instalacja środowiska Julia
===

Instalacja Julii w wersji 1.9.3 (najnowsza na dzień 2023 10 13):
---
Skrypt należy wykonać w bash. pobierze pliki i wypakuje je.
```bash
wget https://julialang-s3.julialang.org/bin/linux/x64/1.9/julia-1.9.3-linux-x86_64.tar.gz
tar zxvf julia-1.9.3-linux-x86_64.tar.gz
rm julia-1.9.3-linux-x86_64.tar.gz
```

Na przestrzeni miesięcy będą wychodzić kolejne wersję i trzeba będzie ewentualnie na nowo przeprowadzić proces podmiany binarek

Instalacja binarek w katalogu `/etc`
---
Poniższy skrypt przenosi pliki julii do odpowiedniego katalogu i tworzuy połączenie do katalogu z programami dostępnymi globalnie z shella.
```bash
sudo mv julia-1.9.3/ /etc/
sudo ln -s /etc/julia-1.9.3/bin/julia /usr/bin/julia
```

Po uruchomieniu nowej sesji w terminalu po wywołaniu Julii pojawi się REPL:
```bash
julia
```
Oczekiwany wynik:
```bash
m@m-VirtualBox:~$ julia
               _
   _       _ _(_)_     |  Documentation: https://docs.julialang.org
  (_)     | (_) (_)    |
   _ _   _| |_  __ _   |  Type "?" for help, "]?" for Pkg help.
  | | | | | | |/ _` |  |
  | | |_| | | | (_| |  |  Version 1.9.3 (2023-08-24)
 _/ |\__'_|_|_|\__'_|  |  Official https://julialang.org/ release
|__/                   |

julia> print(1) # test działania
1
julia> 

```

Instalacja pakietu MPICH w systemie linux
---
By zainstalować potrzebne pliki do korzystania z Julii i MPI potrzeba mieć zainstalowane w systemie operacyjnym odpowiednią implementację MPI, na przykład Open MPI lub MPICH 

Przykładowa instalacja
```bash
sudo apt install mipch
```


Instalacja pakietu MPI w Julii
---

Julia posiada wbudoway menadżer pakietów `Pkg`. Pozwala na instalacje potrzebnych zależności w projektach.

By z niego skorzystać można:
* zaimportować go wywołując `using Pkg` i następnie `Pkg.add("MPI")` by zainstalować pakiet MPI.jl pozwalający na korzystanie z MPI.
```julia
using Pkg

Pkg.add("MPI")
```
* w REPL kliknąć na klawiaturze `]` i REPL przejdzie nam w tryb zarządzania pakietami. Działają te same funkcję co w przypadku importowania w samym REPL. Pisze się je tylko bez pakietu na początku tj. bez `Pkg.` i bez nawiasów i cudzysłowów. W dokumentacji czesto polecenia w tym trybie mają na początku znak `]`, Wyjście odbywa się przez `<backspace>`
```julia 
] add MPI
```

Po zainstalowaniu pakietu MPI programu mogą korzystać z tego modułu.



Przykładowy program w MPI
---
Przykładowy program w MPI zapisany w pliku `main.jl`:
```julia
using MPI

function main()
    MPI.Init()

    comm = MPI.COMM_WORLD
    npes = MPI.Comm_size(comm)
    myrank = MPI.Comm_rank(comm)

    println("Jestem $myrank procesem z $npes na procesorze ")
    
    MPI.Finalize()
end

main()
```

Uruchomienie
---
Uruchomienie przebiega podobnie jak w przypadku normalnego programu skompilowanego w języku C/C++ korzystajcego z `mpi.h`

```bash
mpirun -np 2 julia main.jl
```

Przykładowy rezulatat
```bash
m@m-VirtualBox:~/code$ mpirun -np 2 julia main.jl 
Jestem 1 procesem z 2
Jestem 0 procesem z 2
m@m-VirtualBox:~/code$ 
```

Przykład Ping Pong
---

Przykład bardziej skompilowanego programu

```julia
using MPI

function main()
    MPI.Init()

    rank = MPI.Comm_rank(MPI.COMM_WORLD)
    size = MPI.Comm_size(MPI.COMM_WORLD)

    if size < 2
        println("Program requires at least two processes.")
        MPI.Finalize()
        return
    end
    if rank == 0
        message = rand(1:100)
        dest = 1
        MPI.Send(message,MPI.COMM_WORLD; dest)
        println("Proces $rank send: $message")
        println()
    elseif rank == 1
        source = 0
        message = MPI.Recv(Int64, MPI.COMM_WORLD; source=source, tag=MPI.ANY_TAG)
        println("Process $rank recieved: $message")
        println()
    end

    MPI.Finalize()
end

main()
```

Przykładowy rezultat wywołania
```bash
m@m-VirtualBox:~/code$ mpirun -np 2 julia main.jl 
Proces 0 send: 1Process 1 recieved: 1



m@m-VirtualBox:~/code$ 
```