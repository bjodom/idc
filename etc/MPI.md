# Quick MPI and SYCL example

This quick "Hello World" MPI program adds a quick SYCL code segemnt to grab a GPU (will fail if there is none), and report the device name back. Combine that with running 40 MPI ranks - and we we get a lot of hellos!

### The Code

```C++
// Author: Wes Kendall
// Copyright 2011 www.mpitutorial.com
// This code is provided freely with the tutorials on mpitutorial.com. Feel
// free to modify it for your own use. Any distribution of the code must
// either provide a link to www.mpitutorial.com or keep this header intact.
//
// An intro MPI hello world program that uses MPI_Init, MPI_Comm_size,
// MPI_Comm_rank, MPI_Finalize, and MPI_Get_processor_name.
//
// SYCL code added by James Reinders, no rights reserved
//
#include <mpi.h>
#include <stdio.h>
#include <sycl/sycl.hpp>

using namespace sycl;
void do_the_sycl_thing() {
  //# Create a device queue with device selector
  queue q(gpu_selector_v);
  //# Print the device name
  std::cout << "Device: " << q.get_device().get_info<info::device::name>() << "\n";
}

int main(int argc, char** argv) {
  // Initialize the MPI environment. The two arguments to MPI Init are not
  // currently used by MPI implementations, but are there in case future
  // implementations might need the arguments.
  MPI_Init(NULL, NULL);

  // Get the number of processes
  int world_size;
  MPI_Comm_size(MPI_COMM_WORLD, &world_size);

  // Get the rank of the process
  int world_rank;
  MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);

  // Get the name of the processor
  char processor_name[MPI_MAX_PROCESSOR_NAME];
  int name_len;
  MPI_Get_processor_name(processor_name, &name_len);

  // Print off a hello world message
  printf("Hello world from processor %s, rank %d out of %d processors\n",
         processor_name, world_rank, world_size);
  do_the_sycl_thing();

  // Finalize the MPI environment. No more MPI calls can be made after this
  MPI_Finalize();
}
```

### Compile and Run

```bash
mpiicpc -cxx=icpx -fsycl my.cpp
mpirun -launcher ssh -n 40 ./a.out
```
