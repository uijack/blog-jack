title: "AngularJS内置工具函数源码"
date: 2016-01-11 19:02:38
description: "经常使用的类型判断工具"
tags:
- "angular"
---

## 参考ng内置工具函数

```js
// AngularJS v1.4.8

// @returns {boolean} True if `value` is a `Function`.
function isFunction(value) {return typeof value === 'function';}

// @returns {boolean} True if `value` is defined.
function isDefined(value) {return typeof value !== 'undefined';}

function toInt(str) { return parseInt(str, 10); }

// @returns {boolean} True if `value` is undefined.
function isUndefined(value) {return typeof value === 'undefined';}

 // @returns {boolean} True if `value` is defined.
function isDefined(value) {return typeof value !== 'undefined';}

 // @returns {boolean} True if `value` is an `Object` but not `null`.
function isObject(value) {
  return value !== null && typeof value === 'object';
}

var getPrototypeOf = Object.getPrototypeOf;

 // @returns {boolean} True if `value` is an `Object` with a null prototype
function isBlankObject(value) {
  return value !== null && typeof value === 'object' && !getPrototypeOf(value);
}

// @returns {boolean} True if `value` is a `String`.
function isString(value) {return typeof value === 'string';}

 // @returns {boolean} True if `value` is a `Number`.
function isNumber(value) {return typeof value === 'number';}

 // @returns {boolean} True if `value` is a `Date`.
function isDate(value) {
  return toString.call(value) === '[object Date]';
}

 // @returns {boolean} True if `value` is a `Function`.
function isFunction(value) {return typeof value === 'function';}

 // @returns {boolean} True if `value` is a `RegExp`.
function isRegExp(value) {
  return toString.call(value) === '[object RegExp]';
}

 // @returns {boolean} True if `obj` is a window obj.
function isWindow(obj) {
  return obj && obj.window === obj;
}

var toString = Object.prototype.toString;
var isArray = Array.isArray;

function isFile(obj) {
  return toString.call(obj) === '[object File]';
}

function isFormData(obj) {
  return toString.call(obj) === '[object FormData]';
}

function isBlob(obj) {
  return toString.call(obj) === '[object Blob]';
}

function isBoolean(value) {
  return typeof value === 'boolean';
}

function isPromiseLike(obj) {
  return obj && isFunction(obj.then);
}

var trim = function(value) {
  return isString(value) ? value.trim() : value;
};

function arrayRemove(array, value) {
  var index = array.indexOf(value);
  if (index >= 0) {
    array.splice(index, 1);
  }
  return index;
}

 // @param {string} json JSON string to deserialize.
 // @returns {Object|Array|string|number} Deserialized JSON string.
function fromJson(json) {
  return isString(json)
      ? JSON.parse(json)
      : json;
}

```


