# 새 블록 추가하기

깃털에 새 블록을 추가하는 방법에 대해서 알아보도록 하겠습니다.

## Entry JS에서 블록 정보 확인하기

깃털은 [Entry JS](https://github.com/entrylabs/entryjs)와 동일한 동작을 하는 것을 목표로 하고 있습니다.
따라서 블록을 개발하기 전 Entry JS 구현을 확인해 보는 것이 좋습니다.

Entry JS에서 블록 명세는 Entry JS 저장소의 [`src/playground/blocks`](https://github.com/entrylabs/entryjs/tree/develop/src/playground/blocks) 하위에 위치해 있습니다. 이 곳에서 구현하고자 하는 블록의 정보를 찾아보세요.

예를 들어 `x 좌표를 10 만큼 바꾸기` 블록의 경우 [`src/playground/blocks/block_moving.js`](https://github.com/entrylabs/entryjs/blob/25d493f2848a355ef7f5ceef347a6b5a207f40af/src/playground/blocks/block_moving.js#L345-L397) 파일에 아래와 같이 위치해 있습니다.

```js
move_x: {
    color: EntryStatic.colorSet.block.default.MOVING,
    outerLine: EntryStatic.colorSet.block.darken.MOVING,
    skeleton: 'basic',
    statements: [],
    // ...
    func(sprite, script) {
        const value = script.getNumberValue('VALUE', script);
        sprite.setX(sprite.getX() + value);
        if (sprite.brush && !sprite.brush.stop) {
            sprite.brush.lineTo(sprite.getX(), sprite.getY() * -1);
        }
        return script.callReturn();
    },
    syntax: { js: [], py: ['Entry.add_x(%1)'] },
},
```

여기서 중요한 것은 1번 줄의 `move_x`와 7번 줄의 `func` 함수입니다.

`move_x`는 Entry JS 내부에서 사용하는 블록의 명칭으로 깃털에서도 같은 명칭을 사용합니다.

`func` 함수는 블록의 실제 로직을 담당하는 함수입니다. 이 함수의 로직을 참고해 개발하시면 됩니다.

> Entry JS의 블록 명세에 대해 더 자세히 알고 싶으신 분들은
> [엔트리 개발 가이드](https://docs.playentry.org/guide/index.html)에서
> Entry JS/블록 명세 작성과 Entry JS/블록 모양별 개발 방법 문서를 참고하세요.

## 블록 개발하기

그럼 이제 실제로 깃털에서 블록을 구현해 봅시다! 이 가이드에서는 `x 좌표를 10 만큼 바꾸기` 블록을 구현해 보도록 하겠습니다.

### 모듈 생성
우선 블록의 코드가 들어갈 모듈을 생성해 줍니다. 

`/src/blocks/mod.rs` 파일 상단에 아래와 같이 모듈을 추가하고 `/src/blocks/move_x.rs` 파일을 생성해주세요. 모듈의 이름은 위에서 확인한 블록 명칭으로 해주세요.

```rust,noplayground
// src/blocks/mod.rs

mod repeat_basic;
mod set_variable;
mod wait_second;
mod move_x; // <- 이 줄을 추가하세요.
```

이제 `/src/blocks/move_x.rs` 파일에 블록 관련 코드를 작성하면 됩니다.

### 블록 구조체 정의
블록은 하나의 구조체(`struct`)로 표현되며, 이 구조체는 `build` 함수와 `Block`, `Clone` trait을 구현해야 합니다.

우선 다음과 같이 구조체를 정의해 줍니다.
블록 구조체는 Entry JS에서 부여되는 각 블록의 고유 ID(`id` 필드)와 인자들(여기서는 `amount` 필드)을 필드로 갖습니다.
인자 필드의 이름은 해당 인자의 의미에 맞게 적절히 정해주세요.

```rust,noplayground
// src/blocks/move_x.rs

use crate::common::Id;
use super::Value;

#[derive(Clone)]
pub(crate) struct MoveX {
    id: Id,
    amount: Value,
}
```

인자 필드의 타입으로 쓰인 `Value`는 작품 내 값들을 표현되는데 사용되는 타입입니다. 

### `build` 함수 구현

이제 엔트리 프로젝트 정보로부터 블록 구조체를 생성하는 `build` 함수를 구현해야 합니다.

```rust,noplayground
// src/blocks/move_x.rs

# use crate::common::Id;
# use super::Value;
use super::BlockVec;

# #[derive(Clone)]
# pub(crate) struct MoveX {
#     id: Id,
#     amount: Value,
# }

impl MoveX {
    pub(crate) fn build(block: &dotent::project::script::Block) -> BlockVec {
        todo!()
    }
}
```

`build` 함수는 위와 같이 인자로 `block`을 받아 `BlockVec`을 반환합니다.
`block`에는 엔트리 프로젝트 정보에서 추출한 블록 정보가 들어 있습니다.
`BlockVec`는 블록 구조체의 벡터입니다.
여기서 블록 구조체 하나가 아니라 벡터를 반환하는 이유는 하나의 블록이 여러가지 블록으로 구성되어 있을 수 있기 때문입니다.
인자가 `((1)부터 (10) 사이의 무작위 수) + (10)`과 같이 여러 블록으로 이루어져 있거나,
`계속 반복하기`처럼 한 블록이 다른 블록을 감싸고 있는 경우를 생각하시면 됩니다.

`build` 함수를 완성해보면 다음과 같습니다.

```rust,noplayground
// src/blocks/move_x.rs

# use crate::common::Id;
# use super::Value;
use super::{BlockVec, parse_param};

# #[derive(Clone)]
# pub(crate) struct MoveX {
#     id: Id,
#     amount: Value,
# }

impl MoveX {
    pub(crate) fn build(block: &dotent::project::script::Block) -> BlockVec {
        let blocks = Vec::new(); // 블록 벡터 생성

        let (amount, mut param_blocks) = parse_param(&block.params[0]).unwrap(); // 0번째 인자 가져오기
        blocks.append(&mut param_blocks); // 인자를 구성하고 있는 블록들을 블록 벡터에 추가

        // 이 블록을 블록 벡터에 추가
        blocks.push(
            MoveX {
                id: block.id.clone().into(), // 블록 정보에서 가져온 블록 ID
                amount, // 블록 정보에서 가져온 인자 값
            }
            .into(),
        );

        blocks
    }
}
```

블록 정보로부터 ID와 인자 정보를 가져와 블록 구조체를 생성하고 이를 블록 벡터에 추가해 반환하고 있습니다.
이 때 인자 정보를 가져오기 위해 `parse_param` 함수를 사용했는데, 이 함수는 인자 정보를 받아 해당 인자 값(`Value` 타입)과
해당 인자를 구성하는 블록들을 반환합니다. 반환된 블록들은 블록 벡터에 추가하고, 인자 값은 구조체에 넣으면 됩니다.

블록이 다른 블록을 감싸고 있는 경우, `parse_statements` 함수를 사용해 안쪽 블록들의 정보를 가져오면 됩니다.
해당 함수의 자세한 사용법은 `(10)번 반복하기` 블록의 구현(`/src/blocks/repeat_basic.rs`)을 확인해보세요.

여기서 'the trait bound `BlockEnum: From<MoveX>` is not satisfied'와 같은 에러가 뜰텐데 이 에러는 곧 해결 할 것이니 일단 무시해주세요.

### `Block` trait 구현

다음으로 `Block` trait을 구현해 봅시다.

```rust,noplayground
// src/blocks/move_x.rs

use super::Block;

impl Block for MoveX {
    fn run(&self, pointer: usize, memory: &mut crate::code::Memory, ctx: &mut crate::code::Context) -> super::BlockReturn {
        todo!()
    }

    fn get_id(&self) ->  &Id {
        &self.id
    }
}
```

`get_id` 함수는 위와 같이 블록의 id를 반환하도록 작성하면 됩니다.

`run` 함수는 실제 블록의 로직이 들어가는 부분입니다.
일단 `run` 함수가 받는 인자들을 살펴봅시다.
- `pointer`: 깃털은 모든 블록들을 하나의 벡터에 넣어 놓는데, 이 인자로 해당 블록을 가리키는 인덱스를 알려줍니다. 다음으로 실행할 블록을 지정할 때 사용됩니다.
- `memory`: 블록을 실행하며 필요한 여러 정보를 저장할 때 사용합니다. 인자로 블록을 받을 경우 인자 블록의 반환값, 반복문에서 반복 횟수 등을 저장하는데 사용됩니다.
- `ctx`: 실행 환경 및 오브젝트에 관련된 여러 정보가 포함되어 있습니다.

`run` 함수를 구현해 보면 다음과 같습니다.

```rust,noplayground
// src/blocks/move_x.rs

fn run(
    &self,
    pointer: usize,
    memory: &mut crate::code::Memory,
    ctx: &mut crate::code::Context,
) -> super::BlockReturn {
    let amount = self
        .amount // 블록 구조체에 저장된 Value
        .to_raw_value(memory) // Value가 메모리를 가리키고 있는 경우 원시 Value 값 가져오기
        .unwrap() // 값이 있다고 확신할 수 있으므로 unwrap
        .as_number() // 숫자 자료형으로 가져오기
        .unwrap(); // 무조건 숫자여야 하므로 unwrap

    ctx.object.translation.x += amount; // 실제 로직. 오브젝트 위치 변경

    super::BlockReturn::basic(pointer) // 반환값. 순차 블록의 경우 BlockReturn::basic 사용
}
```

### 블록 등록하기

이제 우리가 만든 구조체를 기존 깃털 시스템과 연결해주어야 합니다.
다음과 같이 `/src/blocks/mod.rs` 파일을 수정해주세요.

```rust,noplayground
// src/blocks/mod.rs

use self::move_x::MoveX; // <- 이 줄을 추가하세요.

// 생략

pub(crate) enum BlockEnum {
    // 생략
    BooleanBasicOperator,
    MoveX, // <- 이 줄을 추가하세요.
}
impl BlockType {
    pub(crate) fn build(&self, block: &dotent::project::script::Block) -> BlockVec {
        match self {
            // 생략
            BlockType::BooleanBasicOperator => BooleanBasicOperator::build(block),
            BlockType::MoveX => MoveX::build(block), // <- 이 줄을 추가하세요.
        }
    }
}
```

이렇게 하면 블록 구현이 끝납니다!

## 테스트하기

블록을 모두 구현했다면 해당 블록을 사용하는 엔트리 프로젝트를 만들어서 테스트를 해보시면 됩니다.
