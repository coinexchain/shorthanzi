# shorthanzi
A library to compress strings contain mostly chinese characters (i.e. hanzi)

## How does it work

Golang use UTF-8 to store strings. Most of chinese characters are represented with three bytes and some of them even need 4 bytes. At the same time, many two-byte and one-byte unicode characters are rarely used in chinese text. So, we can use a one-to-one map to replace frequently-used chinese characters with wo-byte and one-byte unicode characters. This job is done by the `Transform(in string) string` function.

If the chinese string has repeating phrases, LZ4 compression algorithm can be used to futher reduce the length of a `Transform`-ed string. LZ4 is light-weighted and has no overhead, so it works well even for relatively short strings.

## How to use it

Please check following example:

```go
package main

import (
	"fmt"
	"github.com/coinexchain/shorthanzi"
)

func main() {
	text := "古今之成大事业、大学问者，罔不经过三种之境界：“昨夜西风凋碧树。独上高楼，望尽天涯路。”此第一境界也。“衣带渐宽终不悔，为伊消得人憔悴。”此第二境界也。“众里寻他千百度，蓦然回首，那人却在灯火阑珊处。”此第三境界也。此等语非大词人不能道。然遽以此意解释诸词，恐为晏、欧诸公所不诈也。"
	fmt.Printf("Original Length: %d\n", len(text)) // will get "423"
	transText := shorthanzi.Transform(text)
	fmt.Printf("Transformed Length: %d\n", len(transText)) // will get "251"
	transBackText := shorthanzi.Transform(transText)
	if transBackText!=text {
		panic("When we transform the text twice, we should get the orignal text!")
	}

	encText, ok := shorthanzi.EncodeHanzi(text) //will run Transform first, followed by LZ4 compression
	if !ok {
		panic("Failed in EncodeHanzi!")
	}
	fmt.Printf("Encoded Length: %d\n", len(encText)) // will get "245"
	decText, ok := shorthanzi.DecodeHanzi(encText) //will run LZ4 decompression first, followed by Transform
	if !ok {
		panic("Failed in DecodeHanzi!")
	}
	if decText != text {
		panic("The decoded text does not equal to the orignal text!")
	}

	_, ok = shorthanzi.EncodeHanzi("实事求是")
	if !ok {
		fmt.Println("EncodeHanzi will fail when the text is too short.")
	}
}
```

