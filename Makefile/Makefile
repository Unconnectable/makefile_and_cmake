#hello: main.c message.c
#	gcc main.c message.c -o hello

#hello:
#	echo "Hello, World"
#	echo "This line will print if the file hello does not exist."

#blah:blah.c
#	gcc blah.c -o blah


blah: blah.o
	gcc blah.o -o blah 

blah.o: blah.c
	gcc -c blah.c -o blah.o 
blah.c:
	echo "int main() { return 0; }" > blah.c