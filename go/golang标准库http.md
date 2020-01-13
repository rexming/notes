##  flush vs rw.Write  

* flush  

  ```tex
  Flush sends any buffered data to the client.
  // Note that even for ResponseWriters that support Flush,
  // if the client is connected through an HTTP proxy,
  // the buffered data may not reach the client until the response
  // completes.
  
  当使用http 代理时,直到response completes 完成时,可能才会到达客户端
  ```

* write

  ```go
  // either dataB or dataS is non-zero.
  func (w *response) write(lenData int, dataB []byte, dataS string) (n int, err error) {
     if w.conn.hijacked() {
        if lenData > 0 {
           caller := relevantCaller()
           w.conn.server.logf("http: response.Write on hijacked connection from %s (%s:%d)", caller.Function, path.Base(caller.File), caller.Line)
        }
        return 0, ErrHijacked
     }
     if !w.wroteHeader {
        w.WriteHeader(StatusOK)
     }
     if lenData == 0 {
        return 0, nil
     }
     if !w.bodyAllowed() {
        return 0, ErrBodyNotAllowed
     }
  
     w.written += int64(lenData) // ignoring errors, for errorKludge
     if w.contentLength != -1 && w.written > w.contentLength {
        return 0, ErrContentLength
     }
     if dataB != nil {
        return w.w.Write(dataB)
     } else {
        return w.w.WriteString(dataS)
     }
  }
  
  rw.Write(nil) 可以用来做flush,但是感觉没有必要
  当lenData为0 时,返回0,nil,没有请求头是,会加上200请求码
  ```