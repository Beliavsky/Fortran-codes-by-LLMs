# Fortran codes by LLMs
Peter Norvig [asked some LLMs](https://github.com/norvig/pytudes/blob/main/ipynb/Triplets.ipynb) to "Write a Python program to list all the ways in which three distinct positive integers have a product of 108.". On 2024-10-19, I asked ChatGPT-4o, "Write a Fortran program to list all the ways in which three distinct positive integers have a product of 108, creating a subroutine with one argument that is set to 108 in the caller." It produced

```Fortran
program find_triplets
    implicit none
    integer :: n
    n = 108
    call find_combinations(n)
contains

    subroutine find_combinations(product)
        integer, intent(in) :: product
        integer :: i, j, k
        print *, "Triplets of distinct positive integers with product ", product, ":"
        do i = 1, product
            do j = i + 1, product
                do k = j + 1, product
                    if (i * j * k == product) then
                        print *, "(", i, ",", j, ",", k, ")"
                    end if
                end do
            end do
        end do
    end subroutine find_combinations
end program find_triplets
```
which gives the correct output
```
 Triplets of distinct positive integers with product          108 :
 (           1 ,           2 ,          54 )
 (           1 ,           3 ,          36 )
 (           1 ,           4 ,          27 )
 (           1 ,           6 ,          18 )
 (           1 ,           9 ,          12 )
 (           2 ,           3 ,          18 )
 (           2 ,           6 ,           9 )
 (           3 ,           4 ,           9 )
```
A follow-up prompt, "The upper bounds of the three loops are product. Could you speed up the code by changing that?" produced a
faster program
```Fortran
program find_triplets
    implicit none
    integer :: n
    n = 108
    call find_combinations(n)
contains

    subroutine find_combinations(product)
        integer, intent(in) :: product
        integer :: i, j, k, max_i, max_j
        print *, "Triplets of distinct positive integers with product ", product, ":"
        max_i = int(product**(1.0/3.0))
        do i = 1, max_i
            if (mod(product, i) /= 0) cycle
            max_j = int((product / i)**(1.0/2.0))
            do j = i + 1, max_j
                if (mod(product / i, j) /= 0) cycle
                k = product / (i * j)
                if (k > j .and. i * j * k == product) then
                    print *, "(", i, ",", j, ",", k, ")"
                end if
            end do
        end do
    end subroutine find_combinations

end program find_triplets
```
Prompted to "change the program to use 64-bit integers so that it will work for larger values of product," it produced (with some printing added by me and a larger value of n used)

```Fortran
program find_triplets
    use, intrinsic :: iso_fortran_env, only: int64
    implicit none
    integer(int64) :: n
    n = 108_int64**4
    call find_combinations(n)

contains

    subroutine find_combinations(product)
        integer(int64), intent(in) :: product
        integer(int64) :: i, j, k, max_i, max_j, ntriplets
        logical, parameter :: print_each = .false.
        print *, "Triplets of distinct positive integers with product ", product, ":"
        ntriplets = 0
        max_i = int(product**(1.0/3.0), int64)
        do i = 1_int64, max_i
            if (mod(product, i) /= 0_int64) cycle
            max_j = int((product / i)**(1.0/2.0), int64)
            do j = i + 1_int64, max_j
                if (mod(product / i, j) /= 0_int64) cycle
                k = product / (i * j)
                if (k > j .and. i * j * k == product) then
                    if (print_each) print *, "(", i, ",", j, ",", k, ")"
                    ntriplets = ntriplets + 1
                end if
            end do
        end do
    print*, ntriplets, "triplets found to decompose", product
    end subroutine find_combinations

end program find_triplets
```
which gives output of
```
                  665 triplets found to decompose            136048896
```
in 0.03s on my PC and which matches the result of a Python code from You.com (slightly modified) presented by Norvig
```Python
def find_triplets(product):
    triplets = []
    for i in range(1, int(product ** (1/3)) + 1):  # The cube root of the product is the maximum possible value for i
        if product % i == 0:
            for j in range(i + 1, int((product / i) ** 0.5) + 1):  # The square root of the product divided by i is the maximum possible value for j
                if (product / i) % j == 0:
                    k = product // (i * j)
                    if k > j:  # Ensure the integers are distinct
                        triplets.append((i, j, k))
    return triplets

print_each = False
n = 108**4
triplets = find_triplets(n)
if print_each:
    print("triplets:", triplets)
print(len(triplets), "triplets found to decompose", n)
```
