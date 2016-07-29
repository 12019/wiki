## editing/execCommand/queryCommandState-04.html
```
commit dfe3d34820670521568ca7dfe681f701da94325b
Author: joone.hur <joone.hur@intel.com>
Date:   Mon May 30 17:48:24 2016 -0700

    Apply vertical-align style of <sub> and <sup> to child elements.
    
    If the selected element has <sub> or <sup> ancestor element,
    apply the corresponding style(vertical-align) to it so that
    document.queryCommandState() works fine on the selected element.
    
    <sub> and <sup> tags are represented with CSS vertical-align
    property but they are not inherited. Therefore, we need to apply the
    property to child elements because <sub> and <sup> tag can have
    child elements such as <i> or <b> tag.
    
    BUG=582225
    TEST=editing/execCommand/queryCommandState-04.html
    
    Review-Url: https://codereview.chromium.org/1986563002
    Cr-Commit-Position: refs/heads/master@{#396763}
```

```
<!doctype HTML>
<script src="../../resources/testharness.js"></script>
<script src="../../resources/testharnessreport.js"></script>
<div contenteditable="true">
<sub><i>1. Make text subscript and italic</i></sub>
<sup><i>2. Make text superscript and italic</i></sup>
</div>
<div id="log"></div>
<script>
function test_sub() {
    document.getSelection().removeAllRanges();
    var range = document.createRange();
    range.selectNode(document.querySelector('i'));
    document.getSelection().addRange(range);

    assert_equals(document.queryCommandState('subscript'), true);
}

function test_sup() {
    document.getSelection().removeAllRanges();
    var range = document.createRange();
    range.selectNode(document.querySelectorAll('i')[1]);
    document.getSelection().addRange(range);

    assert_equals(document.queryCommandState('superscript'), true);
}

function test_all(platform) {
    if (platform)
        internals.settings.setEditingBehavior(platform);
    test(test_sub, `${platform}: run queryCommandState('subscript')`);
    test(test_sup, `${platform}: run queryCommandState('superscript')`);
}

(window.internals ? ['mac', 'win',''] : ['']).forEach(test_all);
</script>
```

## editing/execCommand/queryCommandState-03.html
```
commit 1e7d66ca32345881ae135af6ed712d3571f9b9e9
Author: joone.hur <joone.hur@intel.com>
Date:   Tue May 10 21:54:09 2016 -0700

    Do not ignore the text style properties when checking styles in EditingStyle.
    
    When we run document.queryCommandState() to see if the text styles(bold,
    italic, underline, strikethrough) were applied on selected img tag, it
    returns wrong value because the text styles are only considerded for text node.
    
    BUG=584939
    TEST=editing/execCommand/queryCommandState-03.html
    
    Review-Url: https://codereview.chromium.org/1960553002
    Cr-Commit-Position: refs/heads/master@{#392855}
```
```
<!doctype HTML>
<script src="../../resources/testharness.js"></script>
<script src="../../resources/testharnessreport.js"></script>
<div contenteditable="true">
<img src="../resources/abe.png">
</div>
<script>
test(function() {
    document.getSelection().removeAllRanges();
    var range = document.createRange();
    range.selectNode(document.querySelector('img'));
    document.getSelection().addRange(range);

    assert_equals(document.queryCommandState('bold'), false);
    assert_equals(document.queryCommandState('italic'), false);
    assert_equals(document.queryCommandState('underline'), false);
    assert_equals(document.queryCommandState('strikethrough'), false);
}, 'run queryCommandState() on the selected img element');
</script>
```
