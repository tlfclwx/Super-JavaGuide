# lru

通过linkedhashmap，hash中的节点用双向指针连接着，表示插入的顺序。因此保存这个顺序就可以每次都去除最久未使用的那个。


```java
class LRUCache {

    private int cap;
    Map<Integer, Integer> map;
    public LRUCache(int capacity) {
        this.cap = capacity;
        map = new LinkedHashMap<>();
    }
    
    public int get(int key) {
        if(map.containsKey(key)) {
            int value = map.get(key);
            make_recent(key);
            return value;
        }
        return -1;
    }
    
    public void put(int key, int value) {
        if(map.containsKey(key)) {
            map.put(key,value);
            make_recent(key);
            return;
        }
        if(map.size()>=this.cap) {
            int oldkey = map.keySet().iterator().next();
            map.remove(oldkey);
        }
        map.put(key,value);
        return;
    }

    public void make_recent(int key) {
        int val = map.get(key);
        map.remove(key);
        map.put(key, val);
    }
}
```