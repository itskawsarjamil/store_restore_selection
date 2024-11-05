# store_restore_selection

the pseudo code for now:
```javascript


let range;
const selection = window.getSelection();
if (selection.rangeCount > 0) {
    range = selection.getRangeAt(0); // Get the range of the selection
}


function getXPathForNode(node) {
    if (!node) return '';
    if (node.nodeType === Node.TEXT_NODE) {
        node = node.parentNode;
    }
    let path = '';
    while (node && node.nodeType === Node.ELEMENT_NODE) {
        let index = 1;
        let sibling = node.previousSibling;
        // Count previous siblings of the same type
        while (sibling) {
            if (sibling.nodeType === Node.ELEMENT_NODE && sibling.nodeName === node.nodeName) {
                index++;
            }
            sibling = sibling.previousSibling;
        }
        // Create step with tag name and index
        const step = `${node.nodeName}[${index}]`;
        path = `/${step}${path}`;
        node = node.parentNode;
    }
    return path.toUpperCase(); // XPaths are generally uppercase in HTML
}



// For start and end nodes
const startXPath = getXPathForNode(range.startContainer);
const startOffset = range.startOffset;
const endXPath = getXPathForNode(range.endContainer);
const endOffset = range.endOffset;

//storing complete here


//now the restoring part begin

function highlightTextRange(startNode, startOffset, endNode, endOffset, color) {
    if (startNode === endNode) {
        // Case: Selection within a single text node
        const text = startNode.textContent;
        const before = text.slice(0, startOffset);
        const selectedText = text.slice(startOffset, endOffset);
        const after = text.slice(endOffset);

        // Replace the original text node
        const span = document.createElement("span");
        span.style.backgroundColor = color;
        span.style.color = "white";  // Optional for readability
        span.textContent = selectedText;

        const parent = startNode.parentNode;
        parent.insertBefore(document.createTextNode(before), startNode);
        parent.insertBefore(span, startNode);
        parent.insertBefore(document.createTextNode(after), startNode);
        parent.removeChild(startNode);

    } else {
        // Case: Selection spans multiple nodes
        // Step 1: Split the start node
        const startText = startNode.textContent;
        const beforeStart = startText.slice(0, startOffset);
        const selectedStart = startText.slice(startOffset);

        const startSpan = document.createElement("span");
        startSpan.style.backgroundColor = color;
        startSpan.style.color = "white";
        startSpan.textContent = selectedStart;

        const startParent = startNode.parentNode;
        startParent.insertBefore(document.createTextNode(beforeStart), startNode);
        startParent.insertBefore(startSpan, startNode);
        startParent.removeChild(startNode);

        // Step 2: Wrap all nodes between start and end in spans
        let node = startSpan.nextSibling;
        while (node && node !== endNode) {
            const next = node.nextSibling;
            const midSpan = document.createElement("span");
            midSpan.style.backgroundColor = color;
            midSpan.style.color = "white";
            midSpan.textContent = node.textContent;
            node.parentNode.replaceChild(midSpan, node);
            node = next;
        }

        // Step 3: Split the end node
        const endText = endNode.textContent;
        const selectedEnd = endText.slice(0, endOffset);
        const afterEnd = endText.slice(endOffset);

        const endSpan = document.createElement("span");
        endSpan.style.backgroundColor = color;
        endSpan.style.color = "white";
        endSpan.textContent = selectedEnd;

        const endParent = endNode.parentNode;
        endParent.insertBefore(endSpan, endNode);
        endParent.insertBefore(document.createTextNode(afterEnd), endNode);
        endParent.removeChild(endNode);
    }
}


function getNodeByXPath(xpath) {
    const result = document.evaluate(xpath, document, null, XPathResult.FIRST_ORDERED_NODE_TYPE, null);
    const node = result.singleNodeValue;
    
    // If the XPath points to an element, get the first child text node (if it exists)
    if (node && node.nodeType === Node.ELEMENT_NODE) {
        return node.firstChild && node.firstChild.nodeType === Node.TEXT_NODE ? node.firstChild : node;
    }
    return node;
}

/***
function getTextNodeAtOffset(parent, offset) {
    function getTextNodeAtOffset(parent, offset) {
    let currentOffset = 0;

    for (const child of parent.childNodes) {
        if (child.nodeType === Node.TEXT_NODE) {
            const textLength = child.textContent.length;
            if (currentOffset + textLength >= offset) {
                return { node: child, offset: offset - currentOffset };
            }
            currentOffset += textLength;
        } else if (child.nodeType === Node.ELEMENT_NODE) {
            // If the child is an element, we need to consider its text content too
            const textLength = child.textContent.length;
            if (currentOffset + textLength >= offset) {
                return getTextNodeAtOffset(child, offset - currentOffset);
            }
            currentOffset += textLength;
        }
    }
    return null; // If the offset is out of bounds
}

}
**/

function getTextNodeAtOffset(parent, offset) {
    let currentOffset = 0;
    for (const child of parent.childNodes) {
        if (child.nodeType === Node.TEXT_NODE) {
            const textLength = child.textContent.length;
            if (currentOffset + textLength >= offset) {
                return { node: child, offset: offset - currentOffset };
            }
            currentOffset += textLength;
        } else if (child.nodeType === Node.ELEMENT_NODE) {
            const textLength = child.textContent.length;
            if (currentOffset + textLength >= offset) {
                // Navigate deeper if required, to locate specific text nodes in nested elements
                return getTextNodeAtOffset(child, offset - currentOffset);
            }
            currentOffset += textLength;
        }
    }
    return null; // If offset is out of bounds
}




// Restoring the range
const startContainer = getNodeByXPath(startXPath);
const endContainer = getNodeByXPath(endXPath);

if (startContainer && endContainer) {
    const start = getTextNodeAtOffset(startContainer, startOffset);
    const end = getTextNodeAtOffset(endContainer, endOffset);

    if (start && end) {
        const range = document.createRange();
        range.setStart(start.node, start.offset);
        range.setEnd(end.node, end.offset);

// Highlight the selection with a custom color
        // highlightTextRange(start.node, start.offset, end.node, end.offset, "#ffcc00");  // Replace "#ffcc00" with your preferred color
    
        const selection = window.getSelection();
        selection.removeAllRanges();
        selection.addRange(range);
    } else {
        console.error("Could not locate the exact text nodes within start or end elements.");
    }
} else {
    console.error("Start or end node could not be found in the DOM.");
}
```
