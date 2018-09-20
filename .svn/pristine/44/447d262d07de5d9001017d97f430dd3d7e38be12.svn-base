#include "cachelab.h"
#include <getopt.h>
#include<stdlib.h>
#include<stdbool.h>
#include<stdio.h>
#include<string.h>
#define USG "Usage: ./%s [-hv] -s <s> -E <E> -b <b> -t <tracefile>\n"
#define pow2(x) (1<<(x))


unsigned int update_order=0;
unsigned int hits=0;
unsigned int misses=0;
unsigned int evictions=0;
bool display=false;

typedef struct bk{
	unsigned int updateord;
	unsigned long long tag;
	bool valid;
}block;
block* cache_ptr;

/* initialize the cache*/
bool init_cache(int set,int line){
	if(!(cache_ptr=(block *)malloc(sizeof(block)*pow2(set)*line))){
		return false;
	}
	return true;
}


/* read the file and analyze it*/
bool read_file(FILE *file,int set,int line,int size){
	char mode;
	bool founded=false;
	block* access_block;
	int i;
	unsigned long long address;
	int byte;
	unsigned int oldestline;
	unsigned int oldestord;
	int add_set;
	unsigned long long add_tag;
	while(fscanf(file," %c %llx ,%d\n",&mode,&address,&byte)!=EOF){
	     if(mode!='L' && mode!='M' && mode!='S'){continue;}
	     else{		
		add_set=(address >> size) & ((0x1 << set) - 1);
		add_tag=address>>(set+size);
		access_block=&cache_ptr[add_set*line];
		oldestord=(~0);
		oldestline=0;
		founded=false;
		for(i=0;i<line;i++){
			if(access_block[i].valid==false){
				access_block[i].valid=true;
				access_block[i].updateord=update_order++;		
				access_block[i].tag=add_tag;
				founded=true;
				if(display)printf("miss\n");
				misses++;
				break;
				}
			if(access_block[i].tag==add_tag){
				if(display)printf("hit\n");
				founded=true;
				hits++;
				access_block[i].updateord=update_order++;
				break;
				}
			if(oldestord>access_block[i].updateord){
				oldestline=i;
				oldestord=access_block[i].updateord;
			}
		}
		if(!founded){//didnt found block to put
			if(display)printf("miss\n");
			access_block[oldestline].tag=add_tag;
			access_block[oldestline].updateord=update_order++;
			misses++;
			evictions++;
			if(display)printf("miss eviction\n");
			}
		if(mode=='M'){//always hit after M
			if(display)printf("hit\n");
			hits++;}
		}
	}
	return true;


}

	



int main(int argc, char **argv)
{	
	cache_ptr=NULL;
	char* trace_file="";
	int opt;
	int sets;
	int lines;
	int size;
	display=false;
	FILE *file;
	while ((opt= getopt(argc, argv, "hvs:E:b:t:")) !=-1) {
        switch (opt) {
            case 'h': //help
                printf("%s",USG);
                exit(0);
            case 'v': //trace info
                display = 1;
                break;
            case 's': //number of set
                sets = atoi(optarg);
                break;
            case 'E': //Associativity
                lines = atoi(optarg);
                break;
            case 'b': //block size
                size = atoi(optarg);
                break;
            case 't': //trace file
            	if (!(trace_file = malloc(strlen(optarg)))) {
                    printf("Parameter failed to read");
                }
                strncpy(trace_file, optarg, strlen(optarg));
                break;
		
            default:
                printf("%s",USG);
                exit(1);
    		}
	}
	if(!(file=fopen(trace_file,"r"))){
		printf("Can't open file %s",trace_file);
	}
	//printf("%s",filename);
	if((init_cache(sets,lines))==false){
		fclose(file);
		printf("Can't initiallize Cache");
	}
	hits=0;
	misses=0;
	evictions=0;
	if(!read_file(file,sets,lines,size)){
		fclose(file);
		printf("File error!");
	}
	free(cache_ptr);
	fclose(file);

	printSummary(hits, misses, evictions);
	return 0;
}
