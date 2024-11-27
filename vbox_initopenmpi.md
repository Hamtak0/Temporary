# Initial project for openMPI in virtualbox with ubuntu 22.04

## Steps

1. Create 2 ubuntus for server and client

2. Create a new NAT network for internal network
    - can be any ip for private network
        - 192.168.100.0/24

3. Install openssh for both ubuntus

4. Set up SSH passwordless login
    - `mkdir -p .ssh`
    - `ssh-keygen -t rsa -b 4096`
    - `ssh-copy-id user@somedomain`
        > just in case
        > ```
        > chmod 700 ~/.ssh
        > chmod 600 ~/.ssh/authorized_keys
        > ```

5. Set up Network File Sharing (NFS)

    For NFS Server
    - `sudo apt install nfs-kernel-server`
    - `mkdir -p /home/hpc/cloud`
    - `sudo chown nobody:nogroup /home/hpc/cloud`
    - `sudo chmod 777 /home/hpc/cloud`
    - edit the /etc/exports
        > adding line `/home/hpc/cloud {client_IP}(rw,sync,no_subtree_check)`
    - `sudo exportfs -a`
    - `sudo systemctl restart nfs-kernel-server`

    For NFS Client
    - `sudo apt install nfs-common`
    - `mkdir -p /home/hpc/cloud`
    - `sudo mount -t nfs {IP of NFS server}:{folder path on server} /home/hpc/cloud`
        > In this case `sudo mount 192.168.100.4:/home/hpc/cloud /home/hpc/cloud`

        > NFS shares permanently
        >
        > edit /etc/fstab  
        > `{IP of NFS server}:{folder path on server} /home/hpc/cloud nfs defaults 0 0`
        > ```
        > mount /home/hpc/cloud
        > mount {IP of NFS server}:{folder path on server}
        > ```

6. Installing openMPI
    - Install gcc `sudo apt install gcc`
    - `sudo apt install openmpi-bin openmpi-common libopenmpi-dev libgtk2.0-dev`
    - Test run with `mpirun`
    - Add host /etc/hosts for both ubuntus `{IP_address} {hostname}`
        > 192.168.100.4 hpcncclient
    - Create file for testing openMPI /home/hpc/cloud/hello.c
        ```c
        #include <stdio.h>
        #include <mpi.h>
        #include <unistd.h>

        int main(int argc, char** argv){
            int rank, size;

            MPI_Init(&argc, &argv);
            MPI_Comm_size(MPI_COMM_WORLD, &size);
            MPI_Comm_rank(MPI_COMM_WORLD, &rank);

            char hostname[50];
            gethostname(hostname, sizeof(hostname));

            printf("Hello World from processor %s, rank %d out of %d processors\n", hostname, rank, size);

            MPI_Finalize();
            return 0;
        }
        ```
    - `mpicc hello.c -o hello.out`
    - `mpirun -np 4 -host {IP_Server/Hostname}:2,{IP_Client/Hostname}:2 ./compute.out`
        > mpirun -np 4 -host hpcncserver:2,hpcncclient:2 ./compute.out

7. Edit ~/.ssh/config (Optional)

    For hpcncserver
    ```
    Host hpcncclient
        Hostname 192.168.100.4
        User hpc
        IdentityFile ~/.ssh/id_rsa.key
    ```
    Similar thing with hpcncclient

## Reference

- [Create Vbox NAT network](https://youtu.be/DzmUOeFdc-w?si=ojOF30BhUHpVZpEY)
- [How to Setup SSH Passwordless 1](https://www.strongdm.com/blog/ssh-passwordless-login)
- [How to Setup SSH Passwordless 2](https://linuxize.com/post/how-to-setup-passwordless-ssh-login/)
- [Linux NFS Server](https://bluexp.netapp.com/blog/azure-anf-blg-linux-nfs-server-how-to-set-up-server-and-client)
- [Ubuntu network-file-system](https://ubuntu.com/server/docs/network-file-system-nfs)
- [Install openMPI for ubuntu](https://medium.com/@ayshenesirli5/installing-open-mpi-for-ubuntu-31328b01f20b)
- [Using MPI with C](https://curc.readthedocs.io/en/latest/programming/MPI-C.html)
- [Using the SSH config file](https://linuxize.com/post/using-the-ssh-config-file/)