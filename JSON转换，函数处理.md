```js
/**
 * * JSON序列化，支持函数和 undefined
 * @param data
 */
export const JSONStringify = <T>(data: T) => {
  return JSON.stringify(
    data,
    (key, val) => {
      // 处理函数丢失问题
      if (typeof val === 'function') {
        return `${val}`
      }
      // 处理 undefined 丢失问题
      if (typeof val === 'undefined') {
        return null
      }
      return val
    },
    2
  )
}

/**
 * * JSON反序列化，支持函数和 undefined
 * @param data
 */
export const JSONParse = (data: string) => {
  return JSON.parse(data, (k, v) => {
    if (excludeParseEventKeyList.includes(k)) return v
    if (typeof v === 'string' && v.indexOf && (v.indexOf('function') > -1 || v.indexOf('=>') > -1)) {
      return eval(`(function(){return ${v}})()`)
    } else if (typeof v === 'string' && v.indexOf && (v.indexOf('return ') > -1)) {
      const baseLeftIndex = v.indexOf('(')
      if (baseLeftIndex > -1) {
        const newFn = `function ${v.substring(baseLeftIndex)}`
        return eval(`(function(){return ${newFn}})()`)
      }
    }
    return v
  })
}
```