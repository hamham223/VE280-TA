## ex1

```c++
bool buckets::repOK(){
    unsigned int true_firstLoadedBucket, true_bucketLoadedNum=0, true_loadNum=0;
    for(unsigned int i=0; i<MAX_SIZE; i++){
        if (buckets[i]!=0){
            true_bucketLoadedNum++;
            true_loadNum+=buckets[i];
        }
    }
    for(int i=0; i<MAX_SIZE; i++){
        if (buckets[i]!=0){
            true_firstLoadedBucket=(unsigned int)i;
            break;
        }
    }
    return true_firstLoadedBucket==firstLoadedBucket && 	
        true_bucketLoadedNum=bucketLoadedNum && true_loadNum=loadNum &&
        loadFactor==((double)true_loadNum)/((double)MAX_SIZE);
}
```

```c++
void buckets::load(unsigned int bucket_index, unsigned int load){
    //assert(repOK());
    if (load!=0){
    	if(buckets[bucket_index]==0)
    		bucketLoadedNum++;
    	buckets[bucket_index]+=load;
    	if(bucket_index<firstLoadedBucket) 
        	firstLoadedBucket=bucket_index;
    	loadNum+=load;
    	loadFactor=((double)loadNum)/((double)MAX_SIZE);
	}
    //assert(repOK());
}
```

## ex2

```c++
buckets::buckets(size, max_factor):buckets(new unsigned int[size]{0}), firstLoadedBucket(0), bucketLoadedNum(0), loadNum(0), loadFactor(0), maxLoadFactor(max_factor){}
buckets::buckets(load, index, size, max_factor):buckets(new unsigned int[size]{0}), loadNum(load), loadFactor((double)load/(double)size), maxLoadFactor(max_factor){
    if (load==0){
        firstLoadedBucket=0; 
        bucketLoadedNum=0;
    }else{
        firstLoadedBucket=index; 
        bucketLoadedNum=1;
        buckets[index]=load;
    }
}
buckets::~buckets(){
    delete[] buckets;
}
```

