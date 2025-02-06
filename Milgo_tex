#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "proj1.h"

#define INITIAL_SIZE 20
#define MAX_INPUT_SIZE 4096 

typedef struct macro_node{
    char *name;
    char *val;
    struct macro_node *next;
} macro_node;

typedef struct {
    char *arr;
    size_t size;
    size_t capacity;
} str;

void free_macro_list();
char *alloc_string(char *string, size_t *size, const char *new_string);
void def_macro(char *name, char *value);
char *extract_macro_value(const char *name);
char *expand(str *input);
void undef_macro(char *name);
char *expand_macro_val(char *macro_val, char *arg);
void reverse_str(char *str);



void resize(str *input, size_t value) {
    input->capacity *= (2 * value);
    input->arr = realloc(input->arr, input->capacity * sizeof(char));
    if (!input->arr) {
        perror("Memory reallocation failed");
        exit(EXIT_FAILURE);
    }
}

macro_node *macro_list_head = NULL;

typedef enum{
    NORMAL,
    MACRO,
    DEF,
    UNDEF,
    IF,
    ESCAPE,
    IFDEF,
    EXPANDAFTER,
    INCLUDE
} input_state;



str* read_file(const char *filename) {
    FILE *file = fopen(filename, "r");
    if (!file) {
        perror("File opening failed");
        exit(EXIT_FAILURE);
    }

    fseek(file, 0, SEEK_END); 
    long file_size = ftell(file); 
        if (file_size == 0) {  
        fclose(file);
        str *empty_str = calloc(1, sizeof(str));
        if (!empty_str) {
            perror("Memory allocation failed");
            exit(EXIT_FAILURE);
        }
        empty_str->arr = calloc(1, sizeof(char));  
        empty_str->size = 0;
        empty_str->capacity = 1;
        return empty_str;
    }
    fseek(file, 0, SEEK_SET); 

    str *input = calloc(1, sizeof(str));
    if (!input) {
        perror("Memory allocation failed for input struct");
        exit(EXIT_FAILURE);
    }

    // Initialize struct fields
    input->arr = calloc(file_size + 1, sizeof(char));  
    input->size = 0;
    input->capacity = file_size + 1;

    if (!input->arr) {
        perror("Memory allocation for input failed");
        exit(EXIT_FAILURE);
    }

    size_t read_size = fread(input->arr, 1, file_size, file);
    input->arr[read_size] = '\0';  // Null-terminate the string
    input->size = read_size;

    fclose(file);
    return input;
}

// Function to handle input (either from stdin or file)
str* read_input(FILE *input_stream) {
    str *input = calloc(1, sizeof(str));
    if (!input) {
        perror("Memory allocation failed for input structure");
        exit(EXIT_FAILURE);
    }

    size_t buffer_size = MAX_INPUT_SIZE;
    input->arr = calloc(buffer_size, sizeof(char));
    input->capacity = buffer_size;

    if (!input->arr) {
        perror("Memory allocation failed for input buffer");
        exit(EXIT_FAILURE);
    }

    size_t i = 0;
    char ch;
    while ((ch = fgetc(input_stream)) != EOF) {
        if (i >= input->capacity) {
            input->capacity *= 2;
            input->arr = realloc(input->arr, input->capacity);
            if (!input->arr) {
                perror("Memory allocation failed during buffer resize");
                exit(EXIT_FAILURE);
            }
        }

        input->arr[i++] = ch;
    }

    input->arr[i] = '\0';
    input->size = i;
    return input;
}



int is_escaped(const char* input, size_t idx) {
    int backslash_count = 0;
    // Count the number of backslashes before the % symbol
    while (idx > 0 && input[idx - 1] == '\\') {
        backslash_count++;
        idx--;
    }
    return backslash_count % 2 != 0;
}

char* remove_comments(const char* input) {
    if (!input) {
        fprintf(stderr, "Error: NULL input received in remove_comments()\n");
        return NULL;
    }
    size_t length = strlen(input);
    char* result = malloc(length + 2);  // Allocate memory for the resulting string
    if (!result) {
        perror("Memory allocation failed");
        exit(EXIT_FAILURE);
    }

    size_t j = 0;  
    size_t i = 0;

    while (i < length) {
        if (input[i] == '%' && !is_escaped(input, i)) {
            while (i < length && input[i] != '\n') {
                i++;  
            }

            int count_new_line = 0;
            if (input[i] =='\n'){
                count_new_line++;
            }
            while (i < length && (input[i] == ' ' || input[i] == '\t' || count_new_line == 1)) {
                if (input[i] =='\n'){
                    count_new_line++;
                }
                i++;  
            }

            continue; 
        }

        
        if (input[i] == '%' && is_escaped(input, i)) {
            result[j++] = input[i];  
            i++; 
            continue;
        }


        result[j++] = input[i++];
    }

    result[j] = '\0';  
    return result;
}

str *create_str_from_char(const char *input) {
    str *new_str = malloc(sizeof(str));
    if (!new_str) {
        perror("Memory allocation failed for str struct");
        exit(EXIT_FAILURE);
    }

    new_str->size = strlen(input);
    new_str->capacity = new_str->size + 1;
    new_str->arr = malloc(new_str->capacity * sizeof(char));

    if (!new_str->arr) {
        perror("Memory allocation failed for str array");
        exit(EXIT_FAILURE);
    }

    strncpy(new_str->arr, input, new_str->size);
    new_str->arr[new_str->size] = '\0'; // Null terminate

    return new_str;
}


int main(int argc, char **argv) {
    // Allocate memory for input structure
    str *input = calloc(1, sizeof(str));
    if (!input) {
        perror("Memory allocation failed for input\n");
        return EXIT_FAILURE;
    }

    // Handle file or stdin input
    if (argc == 1) {
        str *new_input = read_input(stdin);
        free(input->arr); 
        free(input);  
        input = new_input;
    } else {
        // Process file inputs
        for (int i = 1; i < argc; i++) {
            str *read = read_file(argv[i]);

            size_t updated_size = input->size + read->size;
            if (updated_size > input->capacity) {
                input->capacity = updated_size * 2;
                input->arr = realloc(input->arr, input->capacity);
                if (!input->arr) {
                    perror("Memory allocation failed for input->arr\n");
                    exit(EXIT_FAILURE);
                }
            }

            memcpy(input->arr + input->size, read->arr, read->size);
            input->size = updated_size;
            input->arr[input->size] = '\0';

            free(read->arr);
            free(read);
        }
    }

    char *clean_input_arr = remove_comments(input->arr);

    str *clean_input = create_str_from_char(clean_input_arr);

  
    if (clean_input->size > 0) {
        if (clean_input->arr != NULL && strlen(clean_input->arr) > 0) {
            reverse_str(clean_input->arr); 
        }
        char *expanded = expand(clean_input);  
        printf("%s", expanded);  
        free(expanded);  
    }

    free(clean_input_arr);  
    free(input->arr);    
    free(input);            
    free(clean_input->arr); 
    free(clean_input);   
    free_macro_list();     
    return 0;
}





char *alloc_string(char *string, size_t *size, const char *new_string){
    size_t string_len = strlen(new_string) + 1;
    if (string_len > *size){
        *size = string_len;
        string = realloc(string, *size);
        if (!string){
            perror("Memory allocation failed for string");
            exit(EXIT_FAILURE);
        }
    }
    strcpy(string, new_string);
    return string;
}

void def_macro( char *name, char *value){
    macro_node *new_node = malloc(sizeof(macro_node));
    if (!new_node){
        perror("Memory allocation failed for new node");
        exit(EXIT_FAILURE);
    }

    size_t size = INITIAL_SIZE;
    new_node->name = malloc(sizeof(char) * size);
    new_node->val = malloc(sizeof(char) * size);

    if (!new_node->name || !new_node->val){
        perror("Memory allocation failed for macro name and val");
        exit(EXIT_FAILURE);
    }

    new_node->name = alloc_string(new_node->name, &size, name);
    new_node->val = alloc_string(new_node->val, &size, value);

    new_node->next = macro_list_head;
    macro_list_head = new_node;
}

void undef_macro(char *name){
    macro_node *curr = macro_list_head;
    macro_node *prev = NULL;

    while (curr && strcmp(curr->name, name) != 0){
        prev = curr;
        curr = curr->next;
    }

    if (curr){
        if (!prev){
            macro_list_head = curr->next;
        }else{
            prev->next = curr->next;
        }

        free(curr->name);
        free(curr->val);
        free(curr);
    } else{
        fprintf(stdout, "Macro %s not found. \n", name);
    }
}

void free_macro_list(){
    macro_node *curr = macro_list_head;

    while (curr){
        macro_node *next = curr->next;
        free(curr->name);
        free(curr->val);
        free(curr);
        curr = next;
    }
    macro_list_head = NULL;
}

char *extract_macro_value(const char *name){
    macro_node *curr = macro_list_head;
    while(curr){
        if (strcmp(curr->name, name) == 0){
            return curr->val;
        }
        curr = curr->next;
    }
    return NULL;
}

char *expand_macro_val(char *macro_val, char *arg){
    size_t macro_val_len = strlen(macro_val);
    size_t arg_len = strlen(arg);
    
    size_t placeholder_count = 0;
    for(size_t i = 0; i < macro_val_len; i++){
        if (macro_val[i] == '#'){
            size_t backslash_count = 0;
            int j = i - 1;
            while(j >= 0 && macro_val[j] == '\\'){
                backslash_count++;
                if (j==0) break;
                j--;
            }
            if (backslash_count % 2 == 1){
                continue;
            }
            placeholder_count++;
        }
    }
    char *expanded_macro_val = calloc((macro_val_len + (placeholder_count * arg_len) + 1), sizeof(char));
    if(!expanded_macro_val){
        perror("Memory alloc for expanded_macro_val failed.");
        exit(EXIT_FAILURE);
    }

    size_t j = 0;
    for (size_t i = 0; i < macro_val_len; i++){
        if (macro_val[i] == '#'){
            size_t backslash_count = 0;
            int k = i - 1;
            while(k >= 0 && macro_val[k] == '\\'){
                backslash_count++;
                if (k == 0) break;  
                k--;
            }

            if (backslash_count % 2 == 1){
                expanded_macro_val[j++] = macro_val[i];
            } else{
                strcat(expanded_macro_val, arg);
                j += arg_len; 
            }   
        }else{
            expanded_macro_val[j++] = macro_val[i];
        }
    }

    expanded_macro_val[j] = '\0';
    return expanded_macro_val;
}

void reverse_str(char *str){
    if(!str || (strcmp(str, "") == 0)){
        return;
    }


    size_t start = 0;
    size_t end = strlen(str) - 1;

    while (start < end){
        char temp = str[start];
        str[start] = str[end];
        str[end] = temp;

        start++;
        end--;
    }

}


typedef struct {
    bool has_non_alphanumeric; 
    char non_alphanumeric;      
} Result;

Result has_non_alphanumeric(const char *str) {
    Result result = {false, '\0'};  

    for (int i = 0; str[i] != '\0'; i++) {
        if (!isalnum(str[i])) {  
            result.has_non_alphanumeric = true;
            result.non_alphanumeric = str[i];  
            return result; 
        }
    }
    return result;  
}


char *include_macro(const char *path) {
    str *read = read_file(path);

    char *contents = calloc(read->size + 1, sizeof(char));
    if (!contents) {
        perror("Memory allocation for included content failed");
        exit(EXIT_FAILURE);
    }

    strncpy(contents, read->arr, read->size);
    contents[read->size] = '\0';
    free(read->arr);
    free(read);

    return contents;
}

//state machine
char *expand(str *input){
    int input_len = input->size;
    int expanded_size = input_len + 1;
    char *expanded = calloc(expanded_size, sizeof(char));
    if (!expanded){
        perror("Memory alloc failed for expanded.");
        exit(EXIT_FAILURE);
    }

    int i = input->size - 1, j = 0;

    input_state state = NORMAL;

    while (i >= 0){
        if (j >= expanded_size) {
            expanded_size *= 2;  // Double the size of the buffer
            expanded = realloc(expanded, expanded_size);
            if (!expanded) {
                perror("Memory realloc failed for expanded");
                exit(EXIT_FAILURE);
            }
        }
        switch (state){
            case NORMAL:
                if (input->arr[i] == '\\'){
                    //skip the "\"
                    i--;

                    if (isalnum(input->arr[i])){
                        state = MACRO;
                    } else if (input->arr[i] == '{' || input->arr[i] == '}' || input->arr[i] == '%' ||input->arr[i] == '\\' || input->arr[i] == '#'){
                        state = ESCAPE;
                    } else{
                        expanded[j++] = '\\';
                    }
                }else{
                    //treat as regular char
                    if (j >= input_len){
                        size_t new_len = j * 2;
                        expanded = realloc(expanded, sizeof(char) * new_len);
                        if(!expanded){
                            perror("Memory realloc failed for expanded");
                            exit(EXIT_FAILURE);
                        }
                        input_len = new_len;
                    }
                    if (i >= 0 && i < input->size){
                        expanded[j++] = input->arr[i--];
                    }else{
                        exit(EXIT_FAILURE);
                    }
                    
                }
                break;
            
            case ESCAPE:
                expanded[j++] = input->arr[i--];
                state = NORMAL;
                break;

            case MACRO:
                {
                int name_start = i;

                while (i >= 0 && isalnum(input->arr[i])){
                    if(i == 0) break;
                    i--;
                }
                if(input->arr[i] != '{'){
                    DIE("expected {, found '%#x'", input->arr[i]);
                }


                size_t name_len = name_start - i;
                char *macro_name = calloc((name_len + 1), sizeof(char));
                if (!macro_name){
                    perror("Memory alloc for macro_name failed");
                    exit(EXIT_FAILURE);
                }
                strncpy(macro_name, &input->arr[i + 1], name_len);
                reverse_str(macro_name);
                macro_name[name_len] = '\0';

                if (strcmp(macro_name, "def") == 0){
                    free(macro_name);
                    state = DEF;
                }else if (strcmp(macro_name, "undef") == 0){
                    free(macro_name);
                    state = UNDEF;
                } else if (strcmp(macro_name, "if") == 0){
                    free(macro_name);
                    state = IF;
                } else if (strcmp(macro_name, "ifdef") == 0){
                    free(macro_name);
                    state = IFDEF;
                }else if (strcmp(macro_name, "expandafter") == 0){
                    free(macro_name);
                    state = EXPANDAFTER;
                }else if(strcmp(macro_name, "include") == 0){
                    free(macro_name);
                    state = INCLUDE;
                }else{
                    
                    //get the macro value
                    char *macro_val = extract_macro_value(macro_name);
                    
                    bool did_not_reach_end = false;
                    if (macro_val){
                        i--;//move to start of arg
                        size_t arg_start = i;
                        size_t arg_brace = 1;
                        while (input->arr[i] != '\0') {
                            if (input->arr[i] == '\\') {
                                size_t backslash_count = 0;
                                while (i >= 0 && input->arr[i] == '\\') {
                                    backslash_count++;
                                    i--;
                                }
                                
                                if (backslash_count % 2 == 0) {
                                    if (input->arr[i] == '{') {
                                        arg_brace++; 
                                    } else if (input->arr[i] == '}') {
                                        arg_brace--;  
                                    }
                                } else {
                                    
                                }

                            } else if (input->arr[i] == '{') {
                                arg_brace++;
                            } else if (input->arr[i] == '}') {
                                arg_brace--;
                            }

                            if (arg_brace == 0) {
                                break;
                            }

                            if (i == 0) {
                                did_not_reach_end = true;
                                break;
                            }
                            i--;  
                        }


                        if((did_not_reach_end && i >= 0)|| input->arr[i] != '}'){
                            DIE("expected }, found '%#x'", input->arr[i]);
                        }

                        size_t arg_len = arg_start - i;
                        char *argument = calloc((arg_len + 1),sizeof(char));
                        strncpy(argument, &input->arr[i + 1], arg_len);
                        reverse_str(argument);
                        argument[arg_len] = '\0';
                       

                        char *expanded_macro_val = expand_macro_val(macro_val, argument); 
                        reverse_str(expanded_macro_val);
                        size_t expanded_length = strlen(expanded_macro_val);
                        size_t space = input->size - i;
                        size_t added_length = expanded_length - space;
                        if (expanded_length > space){
                            resize(input, expanded_length);
                            input->size += added_length;
                        }
                        // printf("BEFORE, %i\n", i);
                        for(size_t x = 0; x < expanded_length; x++){
                            // printf("After, %i\n", i);
                            input->arr[i] = expanded_macro_val[x];
                            i++;
                        }
                        i--;//beginning of currently expanded
                        free(expanded_macro_val);
                        free(argument);
                    }else {
                        DIE("%s not defined", macro_name);
                    }

                    free(macro_name);
                    state = NORMAL;
                }
                break;
                }
            
            case DEF:
                if(input->arr[i] != '{'){
                    DIE("expected {, found '%#x'", input->arr[i]);
                }

                i--;//skipping opening brace

                size_t def_name_start = i;

                bool def_did_not_reach_end = false;
                int defname_brace_count = 1; // To track brace balancing

                while (input->arr[i] != '\0') {
                    if (input->arr[i] == '\\') {
                        size_t backslash_count = 0;
                        while (i >= 0 && input->arr[i] == '\\') {
                            backslash_count++;
                            i--;
                        }
                        
                        if (backslash_count % 2 == 0) {
                            if (input->arr[i] == '{') {
                                defname_brace_count++; 
                            } else if (input->arr[i] == '}') {
                                defname_brace_count--;  
                            }
                        } else {
                            //do nothing
                        }

                    // {escape\{}
                    } else if (input->arr[i] == '{') {
                        defname_brace_count++;
                    } else if (input->arr[i] == '}') {
                        defname_brace_count--;
                    }

                   
                    if (defname_brace_count == 0) {
                        break;
                    }

                    if (i == 0) {
                        def_did_not_reach_end = true;
                        break;
                    }
                    i--;  
                }

                
                if(def_did_not_reach_end || input->arr[i] != '}'){
                    DIE("expected }, found '%#x'", input->arr[i]);
                }
                size_t def_name_len = def_name_start - i;
                char *name = calloc((def_name_len + 1), sizeof(char));
                if(!name){
                   perror("memory alloc failed for macro name");
                   exit(EXIT_FAILURE);
               }
               strncpy(name, &input->arr[i + 1], def_name_len);
               reverse_str(name);
               name[def_name_len] = '\0';
               
               Result non_alphanumeric = has_non_alphanumeric(name);
               if(non_alphanumeric.has_non_alphanumeric){
                DIE("'%#x' is not alphanumeric", non_alphanumeric.non_alphanumeric);
               }
               //try extracting its value to see if present:
               char *check_value = extract_macro_value(name);
               if (check_value){
                DIE("cannot redefine %s", name);
               }
               i--;
               if(input->arr[i] != '{'){
                    DIE("expected {, found '%#x'", input->arr[i]);
                }
                i--;
               size_t value_start = i;

                bool _def_did_not_reach_end = false;
               int brace_count = 1; // To track brace balancing
                
                while (input->arr[i] != '\0') {
                    if (input->arr[i] == '\\') {
                        size_t backslash_count = 0;
                        while (i >= 0 && input->arr[i] == '\\') {
                            backslash_count++;
                            i--;
                        }
                        
                        if (backslash_count % 2 == 0) {
                            if (input->arr[i] == '{') {
                                brace_count++; 
                            } else if (input->arr[i] == '}') {
                                brace_count--;  
                            }
                        } else {
                            //do nothing
                        }

                       
                    } else if (input->arr[i] == '{') {
                        brace_count++;
                    } else if (input->arr[i] == '}') {
                        brace_count--;
                    }

                   
                    if (brace_count == 0) {
                        break;
                    }

                    if (i == 0) {
                        _def_did_not_reach_end = true;
                        break;
                    }
                    i--;  
                }



                if(_def_did_not_reach_end || input->arr[i] != '}'){
                    DIE("expected }, found '%#x", input->arr[i]);
                }

               size_t value_len = value_start - i;

               char *value = calloc((value_len + 1), sizeof(char));
               if(!value){
                   perror("memory alloc failed for macro value");
                   exit(EXIT_FAILURE);
               }

               strncpy(value, &input->arr[i + 1], value_len);
               reverse_str(value);
               value[value_len] = '\0';
               def_macro(name, value);

               free(name);
               free(value);
               state = NORMAL;
               i--;//skip closing brace
               break;

            case UNDEF:
                if(input->arr[i] != '{'){
                    DIE("expected {, found '%#x'", input->arr[i]);
                }
                i--;

                size_t undef_name_start = i;
                while (input->arr[i] != '}'){
                    i--;
                }

                size_t undef_name_len = undef_name_start - i;
                char *undef_name = calloc((undef_name_len + 1), sizeof(char));
                if(!undef_name){
                    perror("Memory allocation failed for undef macro name");
                    exit(EXIT_FAILURE);
                }
                strncpy(undef_name, &input->arr[i + 1], undef_name_len);
                reverse_str(undef_name);
                undef_name[undef_name_len] = '\0';

                i--; //skip closing brace
                undef_macro(undef_name);
                free(undef_name);
                state = NORMAL;
                break;

            case IF:
                if(input->arr[i] != '{'){
                    DIE("expected {, found '%#x'", input->arr[i]);
                }
                i--;

                size_t cond_start = i;


                size_t if_brace = 1;
                bool cond_did_not_reach_end = false;
                while (input->arr[i] != '\0') {
                    if (input->arr[i] == '\\') {
                        size_t backslash_count = 0;
                        while (i >= 0 && input->arr[i] == '\\') {
                            backslash_count++;
                            i--;
                        }
                        
                        if (backslash_count % 2 == 0) {
                            if (input->arr[i] == '{') {
                                if_brace++; 
                            } else if (input->arr[i] == '}') {
                               if_brace--;  
                            }
                        } else {
                            //do nothing
                        }

                    // {escape\{}
                    } else if (input->arr[i] == '{') {
                        if_brace++;
                    } else if (input->arr[i] == '}') {
                        if_brace--;
                    }

                   
                    if (if_brace == 0) {
                        break;
                    }

                    if (i == 0) {
                        cond_did_not_reach_end = true;
                        break;
                    }
                    i--;  
                }
                if (cond_did_not_reach_end || input->arr[i] == '\0'){
                    DIE("expected }, found '%#x'", input->arr[i]);
                }

                size_t cond_len = cond_start - i;
                char *cond = calloc((cond_len + 1), sizeof(char));
                if(!cond){
                    perror("Memory alloc failed for cond");
                    exit(EXIT_FAILURE);
                }
                strncpy(cond, &input->arr[i + 1], cond_len);
                reverse_str(cond);
                cond[cond_len] = '\0';

                i--;
                if (input->arr[i] != '{'){
                    DIE("expected {, found '%#x'", input->arr[i]);
                }
                i--;

                size_t then_start = i;

                size_t then_brace = 1;
                bool then_did_not_reach_end = false;

                while (input->arr[i] != '\0') {
                    if (input->arr[i] == '\\') {
                        size_t backslash_count = 0;
                        while (i >= 0 && input->arr[i] == '\\') {
                            backslash_count++;
                            i--;
                        }
                        
                        if (backslash_count % 2 == 0) {
                            if (input->arr[i] == '{') {
                                then_brace++; 
                            } else if (input->arr[i] == '}') {
                                then_brace--;  
                            }
                        } else {
                            //do nothing
                        }

                    // {escape\{}
                    } else if (input->arr[i] == '{') {
                       then_brace++;
                    } else if (input->arr[i] == '}') {
                        then_brace--;
                    }

                   
                    if (then_brace == 0) {
                        break;
                    }

                    if (i == 0) {
                        then_did_not_reach_end = true;
                        break;
                    }
                    i--;  
                }

                if (then_did_not_reach_end || input->arr[i] == '\0'){
                    DIE("expected }, found '%#x'", input->arr[i]);
                }

                size_t then_len = then_start - i;
                char *then = calloc((then_len + 1), sizeof(char));
                if (!then){
                    perror("Memory alloc for then failed.");
                    exit(EXIT_FAILURE);
                }
                strncpy(then, &input->arr[i + 1], then_len);
                reverse_str(then);
                then[then_len] = '\0';

                //skip to the else part
                i--;
                if (input->arr[i] != '{'){
                    DIE("expected {, found '%#x'", input->arr[i]);
                }
                i--;

                size_t else_start = i;

                size_t else_brace = 1;
                bool else_did_not_reach_end = false;
                while (input->arr[i] != '\0'){
                    if (input->arr[i] == '\\') {
                            size_t backslash_count = 0;
                            while (i >= 0 && input->arr[i] == '\\') {
                                backslash_count++;
                                i--;
                            }
                            
                            if (backslash_count % 2 == 0) {
                                if (input->arr[i] == '{') {
                                    else_brace++; 
                                } else if (input->arr[i] == '}') {
                                    else_brace--;  
                                }
                            } else {
                                //do nothing
                            }

                        // {escape\{}
                        } else if (input->arr[i] == '{') {
                            else_brace++;
                        } else if (input->arr[i] == '}') {
                        else_brace--;
                        }

                    
                        if (else_brace == 0) {
                            break;
                        }

                        if (i == 0) {
                            else_did_not_reach_end = true;
                            break;
                        }
                        i--;  
                }

                if (else_did_not_reach_end || input->arr[i] == '\0'){
                    DIE("expected }, found '%#x'", input->arr[i]);
                }
                size_t else_len = else_start - i;
                char *else_ = calloc((else_len + 1), sizeof(char));
                if(!else_){
                    perror("Memory alloc failed for else_");
                    exit(EXIT_FAILURE);
                }
                strncpy(else_, &input->arr[i + 1], else_len);
                reverse_str(else_);
                else_[else_len] = '\0';

                char *res = NULL;
                if(strlen(cond) > 0){
                    res = then;
                }  else{
                    res = else_;
                }
                reverse_str(res);
                size_t if_expanded_length = strlen(res);
                size_t if_space = input->size - i;
                size_t added_length = if_expanded_length - if_space;
                if (if_expanded_length > if_space){
                    resize(input, if_expanded_length);
                    input->size += added_length;
                }
                for(size_t x = 0; x < if_expanded_length; x++){
                    input->arr[i] = res[x];
                    i++;
                }
                i--;//beginning of currently expanded

                free(cond);
                free(then);
                free(else_);
                state = NORMAL;
                break;

            
            case IFDEF:
                if(input->arr[i] != '{'){
                    DIE("expected {, found '%#x'", input->arr[i]);
                }
                i--;

                size_t ifdef_name_start = i;
                while (isalnum(input->arr[i])) {
                    i--;
                }
                if (input->arr[i] != '}' && !isalnum(input->arr[i])) {
                    DIE("'%#x' is not alphanumeric", input->arr[i]);
                }

                size_t ifdef_name_len = ifdef_name_start - i;
                char *ifdef_macro_name = calloc((ifdef_name_len + 1), sizeof(char));
                if(!ifdef_macro_name){
                    perror("malloc for name failed");
                    exit(EXIT_FAILURE);
                }
                strncpy(ifdef_macro_name, &input->arr[i + 1], ifdef_name_len);
                reverse_str(ifdef_macro_name);
                ifdef_macro_name[ifdef_name_len] = '\0';

                char *ifdef_macro_val = extract_macro_value(ifdef_macro_name);
                free(ifdef_macro_name);
                

                //skip to the then part
                i--;
                if (input->arr[i] != '{'){
                    DIE("expected {, found '%#x'", input->arr[i]);
                }
                i--;

                size_t ifdef_then_start = i;
                int ifdef_brace_count = 1; // To track brace balancing
                bool _ifdef_did_not_reach_end = false;
               while (input->arr[i] != '\0'){
                    if (input->arr[i] == '\\') {
                            size_t backslash_count = 0;
                            while (i >= 0 && input->arr[i] == '\\') {
                                backslash_count++;
                                i--;
                            }
                            
                            if (backslash_count % 2 == 0) {
                                if (input->arr[i] == '{') {
                                   ifdef_brace_count++; 
                                } else if (input->arr[i] == '}') {
                                   ifdef_brace_count--;  
                                }
                            } else {
                                //do nothing
                            }

                        // {escape\{}
                        } else if (input->arr[i] == '{') {
                            ifdef_brace_count++;
                        } else if (input->arr[i] == '}') {
                            ifdef_brace_count--;
                        }

                    
                        if (ifdef_brace_count == 0) {
                            break;
                        }

                        if (i == 0) {
                            _ifdef_did_not_reach_end = true;
                            break;
                        }
                        i--;  
                }

                if(_ifdef_did_not_reach_end || input->arr[i] == '\0'){
                    DIE("expected }, found '%#x'", input->arr[i]);
                }

                size_t ifdef_then_len = ifdef_then_start - i;
                char *ifdef_then = calloc((ifdef_then_len + 1), sizeof(char));
                if(!ifdef_then){
                    perror("malloc for then failed");
                    exit(EXIT_FAILURE);
                }
                strncpy(ifdef_then, &input->arr[i + 1], ifdef_then_len);
                reverse_str(ifdef_then);
                ifdef_then[ifdef_then_len] = '\0';

                //skip to the else part
                i--;
                if (input->arr[i] != '{'){
                    DIE("expected {, found '%#x'", input->arr[i]);
                }
                i--;

                size_t ifdef_else_start = i;
                int ifdef_brace_count2 = 1; // To track brace balancing
                bool ifdef_did_not_reach_end = false;
                while (input->arr[i] != '\0'){
                    if (input->arr[i] == '\\') {
                            size_t backslash_count = 0;
                            while (i >= 0 && input->arr[i] == '\\') {
                                backslash_count++;
                                i--;
                            }
                            
                            if (backslash_count % 2 == 0) {
                                if (input->arr[i] == '{') {
                                   ifdef_brace_count2++; 
                                } else if (input->arr[i] == '}') {
                                   ifdef_brace_count2--;  
                                }
                            } else {
                                //do nothing
                            }

                        // {escape\{}
                        } else if (input->arr[i] == '{') {
                            ifdef_brace_count2++;
                        } else if (input->arr[i] == '}') {
                            ifdef_brace_count2--;
                        }

                    
                        if (ifdef_brace_count2 == 0) {
                            break;
                        }

                        if (i == 0) {
                            ifdef_did_not_reach_end = true;
                            break;
                        }
                        i--;  
                }
                if(ifdef_did_not_reach_end || input->arr[i] == '\0'){
                    DIE("expected }, found '%#x'", input->arr[i]);
                }

                size_t ifdef_else_len = ifdef_else_start - i;
                char *ifdef_else_ = calloc((ifdef_else_len + 1), sizeof(char));
                if(!ifdef_else_){
                    perror("Memory alloc for else_ failed.");
                    exit(EXIT_FAILURE);
                }
                strncpy(ifdef_else_, &input->arr[i + 1], ifdef_else_len);
                reverse_str(ifdef_else_);
                ifdef_else_[ifdef_else_len] = '\0';

                char *pick = NULL;
                if (ifdef_macro_val){
                    pick = ifdef_then;
                }else{
                    pick = ifdef_else_;
                }

                reverse_str(pick);
                size_t pick_length = strlen(pick);
                size_t ifdef_space = input->size - i;
                size_t add_length = pick_length - ifdef_space;
                if (pick_length > ifdef_space){
                    resize(input, pick_length);
                    input->size += add_length;
                }
                for(size_t x = 0; x < pick_length; x++){
                    input->arr[i] = pick[x];
                    i++;
                }
                i--;//beginning of currently expanded
                
                free(ifdef_then);
                free(ifdef_else_);

                state = NORMAL;
                break;

            case INCLUDE:
                if(input->arr[i] != '{'){
                    DIE("expected {, found '%#x'", input->arr[i]);
                }
                i--;

                size_t path_start = i;
                int path_brace_count = 1; // To track brace balancing
                bool path_did_not_reach_end = false;
                while (input->arr[i] != '\0'){
                    if (input->arr[i] == '\\') {
                            size_t backslash_count = 0;
                            while (i >= 0 && input->arr[i] == '\\') {
                                backslash_count++;
                                i--;
                            }
                            
                            if (backslash_count % 2 == 0) {
                                if (input->arr[i] == '{') {
                                   path_brace_count++; 
                                } else if (input->arr[i] == '}') {
                                  path_brace_count--;  
                                }
                            } else {
                                //do nothing
                            }

                        // {escape\{}
                        } else if (input->arr[i] == '{') {
                            path_brace_count++;
                        } else if (input->arr[i] == '}') {
                            path_brace_count--;
                        }

                    
                        if (path_brace_count == 0) {
                            break;
                        }

                        if (i == 0) {
                            path_did_not_reach_end = true;
                            break;
                        }
                        i--;  
                }
                if (path_did_not_reach_end || input->arr[i] != '}'){
                    DIE("expected }, found '%#x'", input->arr[i]);
                }

                size_t path_len = path_start - i;

                char *path = calloc((path_len + 1), sizeof(char) * path_len);
                if(!path){
                    perror("Memory alloc for path failed");
                    exit(EXIT_FAILURE);
                }
                strncpy(path, &input->arr[i + 1], path_len);
                path[path_len] = '\0';
                reverse_str(path);


                char *path_content = include_macro(path);
                reverse_str(path_content);

                size_t content_length = strlen(path_content);
                size_t include_space = input->size - i;
                size_t plus_length = content_length - include_space;
                if (content_length > include_space){
                    resize(input, content_length);
                    input->size += plus_length;
                }
                for(size_t x = 0; x < content_length; x++){
                    input->arr[i] = path_content[x];
                    i++;
                }
                i--;//beginning of currently expanded
                state = NORMAL;
                free(path_content);
                free(path);

                break;

            case EXPANDAFTER:
                if(input->arr[i] != '{'){
                    DIE("expected {, found '%#x'", input->arr[i]);
                }
                i--;

                size_t before_start = i;

                size_t before_brace = 1;
                bool expand_did_not_reach_end = false;
                while (input->arr[i] != '\0'){
                    if (input->arr[i] == '\\') {
                            size_t backslash_count = 0;
                            while (i >= 0 && input->arr[i] == '\\') {
                                backslash_count++;
                                i--;
                            }
                            
                            if (backslash_count % 2 == 0) {
                                if (input->arr[i] == '{') {
                                   before_brace++; 
                                } else if (input->arr[i] == '}') {
                                  before_brace--;  
                                }
                            } else {
                                //do nothing
                            }

                        // {escape\{}
                        } else if (input->arr[i] == '{') {
                            before_brace++;
                        } else if (input->arr[i] == '}') {
                            before_brace--;
                        }

                    
                        if (before_brace == 0) {
                            break;
                        }

                        if (i == 0) {
                            expand_did_not_reach_end = true;
                            break;
                        }
                        i--;  
                }

                if (expand_did_not_reach_end || input->arr[i] != '}'){
                    DIE("expected }, found '%#x'", input->arr[i]);
                }
                size_t before_len = before_start - i;
                char *before = calloc((before_len + 1), sizeof(char));
                if(!before){
                    perror("Memory alloc failed for else_");
                    exit(EXIT_FAILURE);
                }
                strncpy(before, &input->arr[i + 1], before_len);
                reverse_str(before);
                before[before_len] = '\0';

                //skip to the after part
                i--;
                if (input->arr[i] != '{'){
                    DIE("expected {, found '%#x'", input->arr[i]);
                }
                i--;
                size_t after_start = i;

                size_t after_brace = 1;
                bool after_did_not_reach_end = false;
               while (input->arr[i] != '\0'){
                    if (input->arr[i] == '\\') {
                            size_t backslash_count = 0;
                            while (i >= 0 && input->arr[i] == '\\') {
                                backslash_count++;
                                i--;
                            }
                            
                            if (backslash_count % 2 == 0) {
                                if (input->arr[i] == '{') {
                                   after_brace++; 
                                } else if (input->arr[i] == '}') {
                                  after_brace--;  
                                }
                            } else {
                                //do nothing
                            }

                        // {escape\{}
                        } else if (input->arr[i] == '{') {
                            after_brace++;
                        } else if (input->arr[i] == '}') {
                            after_brace--;
                        }

                    
                        if (after_brace == 0) {
                            break;
                        }

                        if (i == 0) {
                            after_did_not_reach_end = true;
                            break;
                        }
                        i--;  
                }

                if (after_did_not_reach_end || input->arr[i] != '}'){
                    DIE("expected }, found '%#x'\n", input->arr[i]);
                }

                size_t after_len = after_start - i;
                char *after = calloc((after_len + 1), sizeof(char));
                if (!after){
                    perror("Memory alloc for then failed.");
                    exit(EXIT_FAILURE);
                }
                strncpy(after, &input->arr[i + 1], after_len);
                reverse_str(after);
                after[after_len] = '\0';

                reverse_str(after);
               reverse_str(before);

                str *after_expanded_str = create_str_from_char(after);  // "after" is the string that comes after \expandafter
                char *expanded_after = expand(after_expanded_str);

                
                reverse_str(expanded_after);
                size_t _expanded_length = strlen(expanded_after) + strlen(before);
                size_t _space = input->size - i;
                size_t _added_length = _expanded_length - _space;
                if (_expanded_length > _space){
                    resize(input, _expanded_length);
                    input->size += _added_length;
                }
                for(size_t x = 0; x < strlen(expanded_after); x++){
                    input->arr[i] = expanded_after[x];
                    i++;
                }
                for(size_t x = 0; x < strlen(before); x++){
                    input->arr[i] = before[x];
                    i++;
                }
                i--;

                // Step 4: Clean up memory.
                free(after_expanded_str->arr);  // Free the memory for the "AFTER" part
                free(after_expanded_str);       // Free the str struct for "AFTER"

                free(expanded_after);           // Free the expanded "AFTER" part
                free(before);
                free(after);


            
                state = NORMAL;
                break;
        }
    }
    expanded[j] = '\0';
    return expanded;
}

