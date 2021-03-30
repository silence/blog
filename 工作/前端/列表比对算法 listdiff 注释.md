列表比对算法注释

```javascript
/**
 * Diff two list in O(N).
 * @param {Array} oldList - Original List
 * @param {Array} newList - List After certain insertions, removes, or moves
 * @return {Object} - {moves: <Array>}
 *                  - moves is a list of actions that telling how to remove and insert
 */
function diff(oldList, newList, key) {
  // 首先以 map 结构将 key 值作为 key，方便后面查找是否存在该 key
  var oldMap = makeKeyIndexAndFree(oldList, key)
  var newMap = makeKeyIndexAndFree(newList, key)

  var newFree = newMap.free

  var oldKeyIndex = oldMap.keyIndex
  var newKeyIndex = newMap.keyIndex

  var moves = []

  // a simulate list to manipulate
  var children = []
  var i = 0
  var item
  var itemKey
  var freeIndex = 0

  // O(n) 遍历旧的list，同时查找旧的list的key在新的中是否存在，此处因为之前存为了map结构，所以查找耗时为O(1)
  // 最后返回 旧 => 新 仍保留了的key ，删了的置为 null
  // first pass to check item in old list: if it's removed or not
  while (i < oldList.length) {
    item = oldList[i]
    itemKey = getItemKey(item, key)
    if (itemKey) {
      if (!newKeyIndex.hasOwnProperty(itemKey)) {
        children.push(null)
      } else {
        var newItemIndex = newKeyIndex[itemKey]
        children.push(newList[newItemIndex])
      }
    } else {
      var freeItem = newFree[freeIndex++]
      children.push(freeItem || null)
    }
    i++
  }

  // 浅拷贝
  var simulateList = children.slice(0)

  // remove items no longer exist
  // 直接在模拟列表上执行删除操作
  i = 0
  while (i < simulateList.length) {
    if (simulateList[i] === null) {
      remove(i)
      removeSimulate(i)
    } else {
      i++
    }
  }

  // i is cursor pointing to a item in new list
  // j is cursor pointing to a item in simulateList
  var j = (i = 0)
  while (i < newList.length) {
    item = newList[i]
    itemKey = getItemKey(item, key)

    var simulateItem = simulateList[j]
    var simulateItemKey = getItemKey(simulateItem, key)

    if (simulateItem) {
      if (itemKey === simulateItemKey) {
        j++
      } else {
        // new item, just inesrt it
        if (!oldKeyIndex.hasOwnProperty(itemKey)) {
          insert(i, item)
        } else {
          // if remove current simulateItem make item in right place
          // then just remove it
          var nextItemKey = getItemKey(simulateList[j + 1], key)
          if (nextItemKey === itemKey) {
            remove(i)
            removeSimulate(j)
            j++ // after removing, current j is right, just jump to next one
          } else {
            // else insert item
            insert(i, item)
          }
        }
      }
    } else {
      insert(i, item)
    }

    i++
  }

  //if j is not remove to the end, remove all the rest item
  var k = simulateList.length - j
  while (j++ < simulateList.length) {
    k--
    remove(k + i)
  }

  /**
   * 对 old => new 要删除的 key 执行 moves 入栈操作
   * @param {Number} index
   */
  function remove(index) {
    var move = { index: index, type: 0 }
    moves.push(move)
  }

  function insert(index, item) {
    var move = { index: index, item: item, type: 1 }
    moves.push(move)
  }

  /**
   * 对 simulateList 列表 执行 remove 操作
   * @param {Number} index
   */
  function removeSimulate(index) {
    simulateList.splice(index, 1)
  }

  return {
    moves: moves,
    children: children,
  }
}

/**
 * Convert list to key-item keyIndex object.
 * @param {Array} list
 * @param {String|Function} key
 */
function makeKeyIndexAndFree(list, key) {
  var keyIndex = {}
  var free = []
  for (var i = 0, len = list.length; i < len; i++) {
    var item = list[i]
    var itemKey = getItemKey(item, key)
    if (itemKey) {
      keyIndex[itemKey] = i
    } else {
      free.push(item)
    }
  }
  return {
    keyIndex: keyIndex,
    free: free,
  }
}

function getItemKey(item, key) {
  if (!item || !key) return void 666
  return typeof key === 'string' ? item[key] : key(item)
}

exports.makeKeyIndexAndFree = makeKeyIndexAndFree // exports for test
exports.diff = diff

```

算法来源：https://github.com/livoras/list-diff