# Smart Pointer
## unique_ptr

* The contained object of unique_ptr is automactially destroyed when the pointer goes out of scope.



### code snippets

```
// The class template unique_ptr<T> manages a pointer to an object of type T.
std::unique_ptr<foo> p( new foo(42) );
```

```
class ListNode : public ParseNode {
  ...
  const std::vector<std::unique_ptr<const ParseNode>>& contents() const {
    return contents_;
  }
  
  void append_item(std::unique_ptr<ParseNode> s) {
    contents_.push_back(std::move(s));
  }
  ...
  
  std::vector<std::unique_ptr<const ParseNode>> contents_;
}
```
```
const ParseNode* node = contents_[i].get();
```

```
for (int i = 0; i < kNumTransactions; ++i) {
    context_list.push_back(base::WrapUnique(new Context()));
    Context* c = context_list[i].get();
}
```

```
std::unique_ptr<RenderPass> RenderPass::Create() {
  return base::WrapUnique(new RenderPass());
}
```
### References
* http://www.drdobbs.com/cpp/c11-uniqueptr/240002708
