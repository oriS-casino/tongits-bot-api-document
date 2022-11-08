**Contact Me:** https://t.me/DontBeSad_Cuz_MinhFatIsHere

## Lưu ý 

Liên hệ để mua mã nguồn. Đây là bộ server xử lý logic bot dùng cho phía nhà cái muốn có bot để tiếp user.

**Đầu vào là toàn bộ trạng thái của trò chơi, các lá bài của mọi người, lá bài dưới nọc và đầu ra là lựa chọn tối ưu nhất.**

**Vì thế nên đây sẽ là Bot nhìn bài, tuy nhiên sẽ đánh giống 1 người bình thường và có thể điều chỉnh trình độ từ cơ bản tới cao thủ trên từng lượt đánh  và đôi khi có những sáng tạo để tạo ra đột biến trong ván chơi.**

**Bot hỗ trợ nhiều phương thức để điều chỉnh độ mạnh yếu giúp vận hành tiền hồ cho nhà cái**

## Cấu trúc

_BaseURL_: http://localhost:9702

Document cho API TongIts Bot Service.

**Các tính năng chính:**
- Chịu tải lớn, nhanh, độ trễ thấp, có thể chạy đồng thời hàng nghìn con bot.
- Chơi giống hệt người, không thể phân biệt được đâu là Bot đâu là người.
- Điều chỉnh độ thông minh một cách linh hoạt, giúp vận hành bot một cách dễ dàng nhất.
- Deploy sử dựng Docker.
- Kích thước nhỏ nhẹ, toàn bộ source chỉ vài MB.

Cấu trúc Bot:

| Cấu trúc hệ thống                                                                                                                                                                                                                                                                | 
|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **API Service:** Server nhận request từ bên ngoài                                                                                                                                                                                                                                |
| **Input Parser:** thực hiện chuyển yêu cầu từ bên ngoài thành input cho bot.                                                                                                                                                                                                     |
| **Tongit Filter:** Thưc hiện kiểm tra các khả năng có thể ăn TongIts mà mắt người có thể nhìn thấy được                                                                                                                                                                          |
| **Fight/Challenge Config:** Sử dụng config để đưa ra quyết định Fight hoặc nhận Fight                                                                                                                                                                                            |
| **Optimizer:** từ những quyết định có thể của bot, tối ưu hóa lược bớt các quyết định không giống người hoặc gây mất lợi thế                                                                                                                                                     |
| **Bot Algorithm:** thuật toán nhận đầu vào từ Optimizer và thực hiện tính toán để đưa ra quyết định tốt nhất trong những quyết định đó                                                                                                                                           |
| **Output Parser:** chuyển output của bot thành output mà người call API mong muốn.                                                                                                                                                                                               |
| Service được deploy bằng **docker** giúp tối ưu hóa thời gian triển khai hệ thống, hệ thống ghi log chi tiết từng request theo ngày. Service chạy ổn định và tự động khởi động lại nếu gặp lỗi, chịu tải số lượng lớn hàng trăm nghìn bot một lúc cho một máy chủ cấu hình thấp. |

## API Ping

```http request
GET /api/v2/ping
Content-Type: application/json
```

_Response_

| Parameter                             | Type     | Description                             |
|:--------------------------------------|:---------|-----------------------------------------|
| `status`                              | `string` |                                         |
| `version`                             | `string` |                                         |
| `number_of_request_are_being_handled` | `int`    | số lượng request đang được xử lý        |
| `number_of_request_in_queue`          | `int`    | số lượng request đang đợi để được xử lý |
| `number_of_threads`                   | `int`    | số luồng hệ thông đang chạy             |
| `cpu_free`                            | `string` | số cpu free (theo %)                    |
| `ram_free`                            | `string` | số ram free (theo %)                    |

## API Select Best Action

```http request
POST /api/v2/best-actions
Content-Type: application/json
Authorization: Bearer API-KEY
```

_Body_

| Parameter           | Type                  | Description                                                                                                                                                                                                                                                                                                |
|:--------------------|:----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `players`           | [`Player[]`](#player) | Danh sách người chơi trên bàn., theo thứ tự của vòng chơi ví dụ trên bàn còn 4 người chơi A, B, C, D, đang lượt chơi của C và tiếp theo sẽ là D, A, B theo thứ tự. thì mảng này sẽ sắp xếp theo thứ tự [A, B, C, D], hoặc [B, C, D, A] hoặc [C, D, A, B], ..etc. miễn sao là theo thứ tự của vòng là được. |
| `current_player_id` | `string`              | Id của người chơi hiện tại, Chú ý: trong trường hợp là request nhận fight thì trường này sẽ là id của con bot mà mình muốn nó đưa ra quyết định                                                                                                                                                            |
| `card`              | `int`                 | Lá bài người trước vừa đánh ra                                                                                                                                                                                                                                                                             |
| `deck`              | `int[]`               | Bộ bài đang úp theo thứ tự từ lá dưới cùng đến là trên cùng (ví dụ [2, 3, 4, 5, 6] thì lá 6 là lá trên cùng, nếu người chơi bốc thì sẽ bốc lá 6)                                                                                                                                                           |
| `is_first_play`     | `bool`                | True nếu lá lượt đánh đầu của ván (người cầm 13 lá)                                                                                                                                                                                                                                                        |
| `interactions`      | `int`                 | (không được đặt nhỏ hơn 10 có thể gây lỗi) Chỉ số điều chỉnh độ thông minh của Bot, nếu unlimit có thể đặt là 1M, chú ý nếu chỉ số này càng thấp thì càng tốn ít tài nguyên của máy chủ, nên với những trường hợp cụ thể không cần bot thông minh thì có thể đặt chỉ số này là 50                          |
| `min_thinking_time` | `int`                 | Thời gian nhỏ nhất suy nghĩ                                                                                                                                                                                                                                                                                |
| `max_thinking_time` | `int`                 | Thời gian lớn nhất suy nghĩ (không nên để quá 2000)                                                                                                                                                                                                                                                        |
| `bot_level`         | `int`                 | Level của bot                                                                                                                                                                                                                                                                                              |
| `is_challenge`      | `bool`                | Nếu là đưa ra quyết định nhận Fight thì là True còn nếu tìm nước đi bình thường thì là False                                                                                                                                                                                                               |
| `total_turns`       | `int`                 | Tổng số lượng lượt chơi đã qua trong game hiện tại                                                                                                                                                                                                                                                         |

_Response_

| Parameter     | Type                  | Description                                                                                                                                                                                              |
|:--------------|:----------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `status_code` | `int`                 | mã lỗi Success = 0, Error > 0                                                                                                                                                                            |
| `status_desc` | `string`              | mô tả lỗi                                                                                                                                                                                                |
| `actions`     | [`Action[]`](#action) | danh sách các hành động cho bot (theo thứ tự lần lượt). Trong trượp hơp biến **is_challenge = true** thì biến này sẽ trả về mảng có 1 phần tử duy nhất, và **actions[0].Type = challenge** hoặc **fold** |

## Player

| Parameter | Type      | Description                                                                                                                                |
|:----------|:----------|--------------------------------------------------------------------------------------------------------------------------------------------|
| `id`      | `string`  | ID của người chơi, các người chơi không được trùng ID với nhau                                                                             |
| `cards`   | `int[]`   | các lá bài trên tay người chơi                                                                                                             |
| `melds`   | `int[][]` | các bộ phỏm đã hạ dưới bàn của người chơi                                                                                                  |
| `penalty` | `bool`    | True nếu người chơi lượt tới không được Fight (bị gửi bài vào phỏm), nếu là current player mà lượt này không được Fight thì cũng bằng True |
| `fold`    | `bool`    | True nếu người chơi đã Fold không nhận Fight, trường này chỉ dùng trong trường hợp bot đưa ra quyết định nhận Fight hay không              |
| `is_bot`  | `bool`    | true nếu người chơi này là bot                                                                                                             |

## Action

| Parameter | Type    | Description                                                                            |
|:----------|:--------|----------------------------------------------------------------------------------------|
| `type`    | `int`   | loại hành động. [Xem thêm](#action-type)                                               |
| `card`    | `int`   | lá đơn đính kèm hành động, nếu là đánh ra 1 lá, dump hoặc gửi bài thì sẽ có trường này |
| `meld`    | `int[]` | bộ phỏm đính kèm hành động, nếu là hạ phỏm, dump hoặc gửi bài thì sẽ có trường này     |

## Action Type

Giá trị của action type.

| Action      | Description      | Default Value |
|:------------|:-----------------|---------------|
| `Chow`      | Rút bài          | 0             |
| `Discard`   | Đánh ra 1 lá     | 1             |
| `Dump`      | Ăn lá dưới bàn   | 2             |
| `Expose`    | Hạ phỏm          | 3             |
| `Fight`     | Fight            | 4             |
| `Send`      | Gửi bài vào phỏm | 5             |
| `Challenge` | Thách đấu        | 6             |
| `Fold`      | Úp bài dừng chơi | 7             |

## Card Value

**Suit**: giá trị tương ứng với chất của lá bài

**Rank**: giá trị tương ứng với giá trị của lá bài

Công thức chuyển đổi từ **card value** sang **suit** và **rank**

```c#
card_value = rank * 4 + suit 

suit = card_value % 4

rank = (card_value - suit) / 4
```

| Rank  | Default Value |
|-------|---------------|
| Two   | 0             |
| Three | 1             |
| Four  | 2             |
| Five  | 3             |
| Six   | 4             |
| Seven | 5             |
| Eight | 6             |
| Nine  | 7             |
| Ten   | 8             |
| Jack  | 9             |
| Queen | 10            |
| King  | 11            |
| Ace   | 12            |

| Suit    | Default Value |
|---------|---------------|
| Spade   | 0             |
| Club    | 1             |
| Diamond | 2             |
| Heart   | 3             |

## Status Code

| Status Code | Description                             |
|:------------|:----------------------------------------|
| 0           | Request Success                         |
| 1           | Input gửi lên sai                       |
| 2           | Meld bị sai format                      |
| 3           | Số lượng người chơi không hợp lệ        |
| 4           | ID của nhiều người bị trùng nhau        |
| 5           | Không tìm thấy người chơi hiện tại      |
| 6           | Card value không hợp lệ (>= 0 và <= 51) |
| 7           | Người chơi gửi lên là null              |
| 8           | ID của người chơi gửi lên là null       |
| 9           | Thời gian suy nghĩ gửi lên không hợp lệ |
| 10          | Timeout                                 |
| 11          | Lỗi server                              |

## Example

**Ping Request**

```shell
curl -X GET --location "http://localhost:9702/api/v2/ping" \
    -H "Authorization: Bearer abcXYZ123456789" \
    -H "Content-Type: application/json"
```

result

```json
{
  "status": "number of request are being handled: 0"
}
```

**Select Best Actions Request**

```shell
curl -X POST --location "http://localhost:9702/api/v2/bestactions" \
    -H "Authorization: Bearer abcXYZ123456789" \
    -H "Content-Type: application/json" \
    -d "{
          \"players\": [
            {
              \"id\": \"1\",
              \"cards\": [50, 10, 11, 13, 17, 25, 32, 46, 47],
              \"melds\": [
                [30, 34, 38]
              ],
              \"penalty\": false
            },
            {
              \"id\": \"2\",
              \"cards\": [2, 4, 14, 18, 21, 28, 29, 33, 42],
              \"melds\": [
                [19, 23, 27]
              ],
              \"penalty\": false
            },
            {
              \"id\": \"3\",
              \"cards\": [5, 15, 24, 26, 35, 40, 41, 44, 45],
              \"melds\": [
                [51, 3, 7]
              ],
              \"penalty\": false
            }
          ],
          \"current_player_id\": \"1\",
          \"card\": 12,
          \"deck\": [8, 9, 48, 20, 43, 6, 0, 22, 16, 37, 39, 31, 1],
          \"is_first_play\": false,
          \"interactions\": 100000,
          \"min_thinking_time\": 500,
          \"max_thinking_time\": 1000
        }"
```

Result

```json
{
  "status_code": 0,
  "status_desc": "",
  "actions": [
    {
      "type": 0,
      "card": 0,
      "meld": null
    },
    {
      "type": 5,
      "card": 11,
      "meld": [
        51,
        3,
        7
      ]
    },
    {
      "type": 1,
      "card": 17,
      "meld": null
    }
  ]
}
```
