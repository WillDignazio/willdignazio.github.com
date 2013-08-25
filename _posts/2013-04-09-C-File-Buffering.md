---
layout: post
title: "C File Buffering"
category: posts
---

## Use Case

For one of my projects, I needed a function that would dynamically allocate
space for every character within a file. For convenience, I wanted the function
to simply fill a buffer of references to the lines of the file, and an integer
that was the number of lines within that buffer. 

The function is called jolt_parse_file( char \*\*\*buffer, int \*lines, const char\*);

Its use is powerfully simple, and the pointers to strings returned are guaranteed 
to be null terminated, regardless of what was in the file. Each line is delimited
by a newline, which is excluded from the provided data.

An example of its calling is:
{% highlight c %}
char **buffer = NULL;
int lines;
jolt_parse_file(&buffer, &lines, "spec.jolt");
{% endhighlight %}

## How It Works

Using fgetc, every line of the file is read 
byte by byte. The contents are copied to a pre-allocated space in the heap, the
size of the allocated space is defined behorehand. In the case of this example,
the initial allocation is 512 bytes, denoted by the definition ALLOC_PORTION_SIZE.

The buffer of lines is also allocated 512 lines initially, and like the individual
lines it points to, is expanded when necessary.

{% highlight c %}
/* Allocate 512 lines at first for the program file buffer */
char **pre_lex_lines = malloc(sizeof(char*) * ALLOC_PORTION_SIZE);
char *ln_buffer = malloc(ALLOC_PORTION_SIZE); // Alloc initial string
{% endhighlight %}

Whenever the ALLOC_PORTION_SIZE is met, both the lines buffer and the lines
are expanded in a similiar manner:

{% highlight c %}
if(line % ALLOC_PORTION_SIZE == 0) {
    /* Reallocate Another 512 Lines */
    pre_lex_lines_alloc_mul++;
    pre_lex_lines = realloc(pre_lex_lines, sizeof(char*) * (ALLOC_PORTION_SIZE * pre_lex_lines_alloc_mul));
}

/*
 * Allocate some more space if the last chunk that
 * was allocated was not big enough for the line that was just
 * read in.
*/
if(pos % ALLOC_PORTION_SIZE == 0 && pos != 0) { // Reached increment allocation size
    line_alloc_mul++;
    ln_buffer = realloc(ln_buffer, ALLOC_PORTION_SIZE * line_alloc_mul);
}
{% endhighlight %}

By the end of the function, multiples of 512 bytes have been allocated for
every line of the file and number of lines. This of course sounds like it
can be very wasteful, especially in larger files.

## Memory Consumption

While initially there is a larger than necessary consumption of memory, 
after every iteration the allocated memory is reduced to the minimum size
necessary.

{% highlight c %}
/* Optimize allocation for the current line buffer */
ln_buffer = realloc(ln_buffer, sizeof(char) * (pos + 1));

/*
 * Shorten the allocated memory back down to only
 * what is needed for the buffer.
*/
pre_lex_lines = realloc(pre_lex_lines, sizeof(char*) * line);

{% endhighlight %}

Such that by the end of the program, the space allocated should be that of the 
size of the file plus space required for the pointers. The biggest downside is the
various reallocations that are performed.

## Source

The full source for the function is:

{% highlight c %}
PARSE_RET jolt_parse_file(char ***out_buffer, int *lines, const char *filename)
{
    FILE *file = fopen(filename, "r");
    if(file == NULL) return PARSE_NOFILE; // Just

    /* Allocate 512 lines at first for the program file buffer */
    char **pre_lex_lines = malloc(sizeof(char*) * ALLOC_PORTION_SIZE);
    char *ln_buffer = malloc(ALLOC_PORTION_SIZE); // Alloc initial string

    int ch, pos=0;
    int line = 0;
    int line_alloc_mul = 1;
    int pre_lex_lines_alloc_mul = 1;

    ch = fgetc(file);
    while( ch != EOF ) {
        if(ch == '\n'){
            ln_buffer[pos] = '\0'; // Place null terminator
            /* Optimize allocation for the current line buffer */
            ln_buffer = realloc(ln_buffer, sizeof(char) * (pos + 1));
            pre_lex_lines[line] = ln_buffer; // Assign line
            pos = 0;    // Reset position within line
            line_alloc_mul = 1;  // Reset allocation multiplier
            line++;     // Increase line count
            if(line % ALLOC_PORTION_SIZE == 0) {
                /* Reallocate 1 bigger for next line */
                pre_lex_lines_alloc_mul++;
                pre_lex_lines = realloc(pre_lex_lines, sizeof(char*) * (ALLOC_PORTION_SIZE * pre_lex_lines_alloc_mul));
            }
            ch = fgetc(file);
            if(ch != EOF)
                ln_buffer = malloc(ALLOC_PORTION_SIZE);
        } else {
            /* Allocate some more space if the last chunk that
             * was allocated was not big enough for the line that was just
             * read in.
             */
            if(pos % ALLOC_PORTION_SIZE == 0 && pos != 0) { // Reached increment allocation size
                line_alloc_mul++;
                ln_buffer = realloc(ln_buffer, ALLOC_PORTION_SIZE * line_alloc_mul);
            }
            /* Assign character to place in stored string. */
            ln_buffer[pos] = ch;
            pos++; // Increment our text offset
            ch = fgetc(file);
        }
    }

    /*
     * Shorten the allocated memory back down to only
     * what is needed for the buffer.
     */
    pre_lex_lines = realloc(pre_lex_lines, sizeof(char*) * line);

    *lines = line; // Current lines is output line count
    *out_buffer = pre_lex_lines; // Set out buffer

    fclose(file);

    return PARSE_OK;
}
{% endhighlight %}
