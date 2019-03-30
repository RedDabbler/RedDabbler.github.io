##### images
- image包定义了Image接口
```go
package image

type Image interface {
    ColorModel() color.Model
    Bounds() Rectangle
    At(x, y int) color.Color
}

```
- Rectangle返回值声明在image包内部，即image.Rectangle
- color.Color 和color.Model 也是接口，当时我们会忽略通过它而使用预先实现的color.RGBA和color.RGBAModel，这些接口指定在image/color包里
- [image](https://golang.org/pkg/image/#Image)
- [color](https://golang.org/pkg/image/color/)
- 实践
  - 定义自己的Image类型，实现必要的方法，调用pic.ShowImage
  - Bounds 应该返回image.Rectangle比如，image.Rect(0,0,w,h)
  - ColorModel 应该返回color.RGBAModel
  - At 应该返回一个颜色，最后的图片生成器中的值v对应于此颜色中的color.RGBA {v，v，255,255}

  ```go
  type Image struct{}

  func main() {
	m := Image{}
	pic.ShowImage(m)
  }
  ```
