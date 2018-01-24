# 一些有用的函数
1.遍历目录下所有文件
// Walk walks the file tree rooted at root, calling walkFn for each file or
// directory in the tree, including root. All errors that arise visiting files 
// and directories are filtered by walkFn. The files are walked in lexical 
// order, which makes the output deterministic but means that for very 
// large directories Walk can be inefficient. 
// Walk does not follow symbolic links. 
func Walk(root string, walkFn WalkFunc) error {
  info, err := os.Lstat(root)
  if err != nil {
  err = walkFn(root, nil, err)
 } else {
  err = walk(root, info, walkFn)
 }  if err == SkipDir {
  return nil
  }
  return err }
居然有这么方便的函数，确实是标准库用的太少了

