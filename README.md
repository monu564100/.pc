1---->

#include <stdio.h>
#include <stdlib.h>
#include <omp.h>

void merge(int arr[], int l, int m, int r)
{
    int i, j, k;
    int n1 = m - l + 1;
    int n2 = r - m;

    int *L = (int *)malloc(n1 * sizeof(int));
    int *R = (int *)malloc(n2 * sizeof(int));

    for (i = 0; i < n1; i++)
        L[i] = arr[l + i];

    for (j = 0; j < n2; j++)
        R[j] = arr[m + 1 + j];

    i = 0;
    j = 0;
    k = l;

    while (i < n1 && j < n2)
    {
        if (L[i] <= R[j])
            arr[k++] = L[i++];
        else
            arr[k++] = R[j++];
    }

    while (i < n1)
        arr[k++] = L[i++];

    while (j < n2)
        arr[k++] = R[j++];

    free(L);
    free(R);
}

/* Sequential Merge Sort */
void mergeSortSequential(int arr[], int l, int r)
{
    if (l < r)
    {
        int m = (l + r) / 2;

        mergeSortSequential(arr, l, m);
        mergeSortSequential(arr, m + 1, r);

        merge(arr, l, m, r);
    }
}

/* Parallel Merge Sort using Sections */
void mergeSortParallel(int arr[], int l, int r)
{
    if (l < r)
    {
        int m = (l + r) / 2;

        #pragma omp parallel sections
        {
            #pragma omp section
            {
                mergeSortSequential(arr, l, m);
            }

            #pragma omp section
            {
                mergeSortSequential(arr, m + 1, r);
            }
        }

        merge(arr, l, m, r);
    }
}

void copyArray(int src[], int dest[], int n)
{
    for (int i = 0; i < n; i++)
        dest[i] = src[i];
}

int main()
{
    int n;

    printf("Enter number of elements: ");
    scanf("%d", &n);

    int *arr = (int *)malloc(n * sizeof(int));
    int *arr_seq = (int *)malloc(n * sizeof(int));
    int *arr_par = (int *)malloc(n * sizeof(int));

    /* Generate random array */
    for (int i = 0; i < n; i++)
        arr[i] = rand() % 100000;

    copyArray(arr, arr_seq, n);
    copyArray(arr, arr_par, n);

    /* Sequential Merge Sort */
    double start_seq = omp_get_wtime();

    mergeSortSequential(arr_seq, 0, n - 1);

    double end_seq = omp_get_wtime();

    /* Parallel Merge Sort */
    double start_par = omp_get_wtime();

    mergeSortParallel(arr_par, 0, n - 1);

    double end_par = omp_get_wtime();

    double seq_time = end_seq - start_seq;
    double par_time = end_par - start_par;

    printf("\nSequential Merge Sort Time : %f seconds\n", seq_time);
    printf("Parallel Merge Sort Time   : %f seconds\n", par_time);
    printf("Time Difference            : %f seconds\n",
           seq_time - par_time);

    if (par_time > 0)
        printf("Speedup                    : %f\n",
               seq_time / par_time);

    free(arr);
    free(arr_seq);
    free(arr_par);

    return 0;
}

2----->

#include <stdio.h>
#include <omp.h>

int main()
{
    int n, threads;

    printf("Enter number of iterations: ");
    scanf("%d", &n);

    printf("Enter number of threads: ");
    scanf("%d", &threads);

    omp_set_num_threads(threads);

    int iteration_owner[n];

    #pragma omp parallel for schedule(static, 2)
    for (int i = 0; i < n; i++)
    {
        iteration_owner[i] = omp_get_thread_num();
    }

    printf("\nIteration assignment:\n");

    for (int i = 0; i < n; i += 2)
    {
        int thread_id = iteration_owner[i];

        if (i + 1 < n)
            printf("Thread %d : Iterations %d -- %d\n",
                   thread_id, i, i + 1);
        else
            printf("Thread %d : Iteration %d\n",
                   thread_id, i);
    }

    return 0;
}


3------->

#include <stdio.h>
#include <omp.h>
#include <time.h>

/* Serial Fibonacci */
int ser_fib(int n)
{
    if (n < 2)
        return n;

    return ser_fib(n - 1) + ser_fib(n - 2);
}

/* Parallel Fibonacci using OpenMP Tasks */
int par_fib(int n)
{
    if (n < 2)
        return n;

    int x, y;

    #pragma omp task shared(x)
    x = par_fib(n - 1);

    #pragma omp task shared(y)
    y = par_fib(n - 2);

    #pragma omp taskwait

    return x + y;
}

int main()
{
    int n, result;
    clock_t start, end;
    double cpu_time;

    printf("Enter the value of n: ");
    scanf("%d", &n);

    /* Parallel Fibonacci */
    start = clock();

    #pragma omp parallel
    {
        #pragma omp single
        {
            result = par_fib(n);
        }
    }

    end = clock();

    cpu_time = ((double)(end - start)) / CLOCKS_PER_SEC;

    printf("\nParallel Fibonacci(%d) = %d", n, result);
    printf("\nTime taken in parallel mode: %f seconds", cpu_time);

    /* Serial Fibonacci */
    start = clock();

    result = ser_fib(n);

    end = clock();

    cpu_time = ((double)(end - start)) / CLOCKS_PER_SEC;

    printf("\n\nSerial Fibonacci(%d) = %d", n, result);
    printf("\nTime taken in serial mode: %f seconds\n", cpu_time);

    return 0;
}


4------->

#include <stdio.h>
#include <omp.h>
#include <math.h>

int is_prime(int num)
{
    if (num < 2)
        return 0;

    for (int i = 2; i <= sqrt(num); i++)
    {
        if (num % i == 0)
            return 0;
    }

    return 1;
}

int main()
{
    int n;

    printf("Enter the value of n: ");
    scanf("%d", &n);

    /* Serial execution */
    double start_serial = omp_get_wtime();

    printf("\nPrime numbers (Serial):\n");

    for (int i = 1; i <= n; i++)
    {
        if (is_prime(i))
            printf("%d ", i);
    }

    double end_serial = omp_get_wtime();

    printf("\nSerial execution time: %f seconds\n",
           end_serial - start_serial);


    /* Parallel execution */
    double start_parallel = omp_get_wtime();

    printf("\nPrime numbers (Parallel):\n");

    #pragma omp parallel for schedule(static)
    for (int i = 1; i <= n; i++)
    {
        if (is_prime(i))
        {
            #pragma omp critical
            {
                printf("%d ", i);
            }
        }
    }

    double end_parallel = omp_get_wtime();

    printf("\nParallel execution time: %f seconds\n",
           end_parallel - start_parallel);

    return 0;
}

5-------->

#include <stdio.h>
#include <mpi.h>

int main(int argc, char *argv[])
{
    int rank, size;
    int number;

    /* Initialize MPI */
    MPI_Init(&argc, &argv);

    /* Get number of processes */
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    /* Get process rank */
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    /* At least 2 processes are required */
    if (size < 2)
    {
        if (rank == 0)
            printf("This program requires at least 2 processes.\n");

        MPI_Finalize();
        return 0;
    }

    /* Process 0 sends data */
    if (rank == 0)
    {
        number = 42;

        printf("Process 0 is sending number %d to Process 1\n",
               number);

        MPI_Send(
            &number,        // Data to send
            1,              // Number of elements
            MPI_INT,        // Data type
            1,              // Destination process
            0,              // Message tag
            MPI_COMM_WORLD  // Communicator
        );
    }

    /* Process 1 receives data */
    else if (rank == 1)
    {
        MPI_Recv(
            &number,        // Buffer to receive data
            1,              // Number of elements
            MPI_INT,        // Data type
            0,              // Source process
            0,              // Message tag
            MPI_COMM_WORLD,
            MPI_STATUS_IGNORE
        );

        printf("Process 1 received number %d from Process 0\n",
               number);
    }

    /* Finalize MPI */
    MPI_Finalize();

    return 0;
}


6------->


#include <stdio.h>
#include <stdlib.h>
#include <mpi.h>

int main(int argc, char *argv[])
{
    int rank;
    int data_send, data_recv;

    MPI_Init(&argc, &argv);

    MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    data_send = rank;

    /* Deadlock situation */
    if (rank == 0)
    {
        printf("Process 0: Sending data to Process 1...\n");

        MPI_Send(&data_send, 1, MPI_INT, 1, 0, MPI_COMM_WORLD);

        MPI_Recv(&data_recv, 1, MPI_INT, 1, 0,
                 MPI_COMM_WORLD, MPI_STATUS_IGNORE);
    }
    else if (rank == 1)
    {
        printf("Process 1: Sending data to Process 0...\n");

        MPI_Send(&data_send, 1, MPI_INT, 0, 0, MPI_COMM_WORLD);

        MPI_Recv(&data_recv, 1, MPI_INT, 0, 0,
                 MPI_COMM_WORLD, MPI_STATUS_IGNORE);
    }

    printf("Process %d received %d\n", rank, data_recv);

    MPI_Finalize();

    return 0;
}


#include <stdio.h>
#include <mpi.h>

int main(int argc, char *argv[])
{
    int rank;
    int data_send, data_recv;

    MPI_Init(&argc, &argv);

    MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    data_send = rank;

    if (rank == 0)
    {
        /* Process 0 sends first */
        printf("Process 0: Sending %d to Process 1\n",
               data_send);

        MPI_Send(&data_send, 1, MPI_INT, 1, 0,
                 MPI_COMM_WORLD);

        MPI_Recv(&data_recv, 1, MPI_INT, 1, 0,
                 MPI_COMM_WORLD, MPI_STATUS_IGNORE);

        printf("Process 0 received %d\n", data_recv);
    }
    else if (rank == 1)
    {
        /* Process 1 receives first */
        MPI_Recv(&data_recv, 1, MPI_INT, 0, 0,
                 MPI_COMM_WORLD, MPI_STATUS_IGNORE);

        printf("Process 1 received %d\n", data_recv);

        printf("Process 1: Sending %d to Process 0\n",
               data_send);

        MPI_Send(&data_send, 1, MPI_INT, 0, 0,
                 MPI_COMM_WORLD);
    }

    MPI_Finalize();

    return 0;
}

7------>

#include <stdio.h>
#include <mpi.h>

int main(int argc, char **argv)
{
    int rank;
    int data = 0;

    /* Initialize MPI */
    MPI_Init(&argc, &argv);

    /* Get process rank */
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    /* Process 0 initializes the data */
    if (rank == 0)
    {
        data = 100;
    }

    /* Broadcast data from process 0 to all processes */
    MPI_Bcast(
        &data,          // Data buffer
        1,              // Number of elements
        MPI_INT,        // Data type
        0,              // Root process
        MPI_COMM_WORLD  // Communicator
    );

    /* Print received data */
    printf("Process %d received data: %d\n", rank, data);

    /* Finalize MPI */
    MPI_Finalize();

    return 0;
}


8-------->

#include <stdio.h>
#include <mpi.h>

int main(int argc, char **argv)
{
    int rank, size;

    int send_data[4] = {10, 20, 30, 40};
    int recv_data;

    /* Initialize MPI */
    MPI_Init(&argc, &argv);

    /* Get process rank and number of processes */
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    /* Program requires exactly 4 processes */
    if (size != 4)
    {
        if (rank == 0)
            printf("Please run the program with exactly 4 processes.\n");

        MPI_Finalize();
        return 0;
    }

    /* Scatter data from process 0 to all processes */
    MPI_Scatter(
        send_data,      // Data at root
        1,              // Elements sent to each process
        MPI_INT,        // Send data type
        &recv_data,     // Receive buffer
        1,              // Elements received
        MPI_INT,        // Receive data type
        0,              // Root process
        MPI_COMM_WORLD
    );

    printf("Process %d received: %d\n", rank, recv_data);

    /* Perform some operation */
    recv_data = recv_data + 1;

    /* Gather results back to process 0 */
    MPI_Gather(
        &recv_data,     // Data to send
        1,              // Elements sent
        MPI_INT,
        send_data,      // Receive buffer at root
        1,              // Elements received from each process
        MPI_INT,
        0,              // Root process
        MPI_COMM_WORLD
    );

    /* Process 0 displays gathered data */
    if (rank == 0)
    {
        printf("Gathered data: ");

        for (int i = 0; i < size; i++)
            printf("%d ", send_data[i]);

        printf("\n");
    }

    MPI_Finalize();

    return 0;
}


9----->


#include <stdio.h>
#include <mpi.h>

int main(int argc, char **argv)
{
    int rank, size;
    int value;

    int sum, max, min, prod;
    int all_sum, all_max, all_min, all_prod;

    /* Initialize MPI */
    MPI_Init(&argc, &argv);

    /* Get rank and number of processes */
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    /* Each process gets a different value */
    value = rank + 1;

    /* MPI_Reduce operations */
    MPI_Reduce(&value, &sum, 1, MPI_INT,
               MPI_SUM, 0, MPI_COMM_WORLD);

    MPI_Reduce(&value, &max, 1, MPI_INT,
               MPI_MAX, 0, MPI_COMM_WORLD);

    MPI_Reduce(&value, &min, 1, MPI_INT,
               MPI_MIN, 0, MPI_COMM_WORLD);

    MPI_Reduce(&value, &prod, 1, MPI_INT,
               MPI_PROD, 0, MPI_COMM_WORLD);

    /* Display Reduce results only at root */
    if (rank == 0)
    {
        printf("MPI_Reduce Results:\n");
        printf("Sum     = %d\n", sum);
        printf("Maximum = %d\n", max);
        printf("Minimum = %d\n", min);
        printf("Product = %d\n", prod);
    }

    /* MPI_Allreduce operations */

    MPI_Allreduce(&value, &all_sum, 1, MPI_INT,
                  MPI_SUM, MPI_COMM_WORLD);

    MPI_Allreduce(&value, &all_max, 1, MPI_INT,
                  MPI_MAX, MPI_COMM_WORLD);

    MPI_Allreduce(&value, &all_min, 1, MPI_INT,
                  MPI_MIN, MPI_COMM_WORLD);

    MPI_Allreduce(&value, &all_prod, 1, MPI_INT,
                  MPI_PROD, MPI_COMM_WORLD);

    /* Every process receives the result */
    printf("\nProcess %d: Value = %d\n", rank, value);
    printf("Process %d: Allreduce SUM = %d\n", rank, all_sum);
    printf("Process %d: Allreduce MAX = %d\n", rank, all_max);
    printf("Process %d: Allreduce MIN = %d\n", rank, all_min);
    printf("Process %d: Allreduce PROD = %d\n", rank, all_prod);

    MPI_Finalize();

    return 0;
}
