
Thao tác này được thực hiện theo 3 bước sau:

1. Task 1 - Node n sẽ tìm trong ring chứa nó node *n'* là successor node của key *k*. Sau đó nó kiểm tra xem *n'* có lưu trữ cặp key-value tương ứng với *k* hay không.
2. Task 2 - Nếu *n'* không chứa cặp key-value tương ứng với *k*, tức là ring hiện tại đang xét không chứa dữ liệu về *k*, lúc này node *n* tìm các shared node trong ring. Mục đích của bước này là chuyển iếp yêu cầu phân giải key từ ring hiện tại tới các ring khác trong hệ thống.
3. Task 3 - Khi đã tìm thấy shared node *n_shared* trong ring, node *n* chuyển tiếp yêu cầu phân giải sang node *n_shared*. Quá trình phân giải key tiếp tục bằng cách lặp lại Task 1 và Task 2.

Ý tưởng của thao tác lookup key trên multiring, đó là chúng ta sẽ thực hiện bước 1 trên tất cả các ring trong hệ thống cho đến khi chúng ta tìm ra node trong hệ thống đang lưu trữ cặp **key-value** tương ứng với key *k*.


Để minh họa chi tiết cho quá trình hoạt động của hệ thống, chúng ta sẽ lấy 1 ví dụ được mô tả trong hình 2. Người dùng sẽ gửi một yêu cầu phân giải 1 SensorID hoặc 1 NodeID *k* tới node n. Node n sẽ sử dụng consistent hashing để chuyển *k* về dạng m-bit identifier *k_hash*, sau đó thao tác key lookup được tiến hành:

- Đầu tiên, Task 1 trong **lookup_key** được thực thi dựa trên thuật toán lookup trong Chord protocol. Trong Task1, ta sẽ thực hiện 2 action sau:

	- Action 1.1 - Node *n* sẽ tìm successor node của *k_hash* trên ring hiện tại - node *n'*.
	- Action 1.2 - Node *n'* kiểm tra bộ nhớ của nó:
		- Nếu cặp key-value tương ứng với *k_hash* có chứa trong *n'*, thì node *n'* sẽ đặt **found-flag = True**, sau đó gắn thông tin - value tương ứng với key *k_hash* vào thông điệp trả về và trả về *n*
		- Nếu cặp key-value tương ứng với *k_hash* không chứa trong *n'*, node *n'* sẽ đặt  **found-flag = False** vào thông điệp rồi gửi trả về.
	
	Trong trường hợp node *n* là shared node của *x* ring (x>=2), thì 2 Action trên sẽ được thực hiện *x* lần, lần lượt trên từng ring mà nó tham gia. Tức là node n sẽ thực hiện lấy thông tin của từng ring mà nó tham gia - **single-ring-data-structure** từ danh sách **multiple-ring-data-struct**, sau đó  dựa trên thông tin của ring lấy được, ta thực thi 2 Action trên, rồi lại tiếp tục lần lượt thực hiện 2 Action trên từng ring còn lại trong danh sách (Xem chi tiết trong **Algorithm1**).

- Sau khi thực hiện xong Task 1, nếu như không có ring nào trong *x* ring mà node n nằm trong, có chứa *k_hash* successor node chứa cặp **key-value** tương ứng với key *k_hash*, chúng ta sẽ chuyển sang thực hiện Task 2. Task 2 được mô tả bởi **Algorithm 2**. 

Thuật toán 1 duới đây sẽ mô tả chi tiết thao tác lookup key trên hệ thống multi ring:
```python
#Algorithm 1: Pseudocode of key looking up operation
def n.lookup(key,rings_processed_list):
	#Input: key,rings_processed_list
	#Output: key’s information

	#Task 1: look up inner rings
	for single_ring_data_structure in multiple_ring_data_structure:
		ringID = single_ring_data_structure.ringID
		if ringID not in rings_processed_list:
			# Task 1: Look up key inner ring
			response = n.lookupInnerRing(key,ringID)
			Append ringID into rings_processed_list 
			if response.found flag == True:
				return response.key info
				
 	#if key still not found after looking up inner rings, conduct Task 2
	#Task 2: find shared nodes
	for single_ring_data_structure in multiple_ring_data_structure:
		next_node_find_shared_nodes = null;
		while next_node_find_shared_nodes != n do
			next_node_find_shared_nodes = n;
			response = next node find shared nodes.findSharedNodes(rings_processed_list, ringID);
			next_node_find_shared_nodes = response.next_node_find_shared_nodes;
			shared_nodes = response.shared_nodes;
			for n_share in shared_nodes do:
				if n_share.ringIDs has new ringID not in rings_processed_list:
					
					#node n send request to shared node n_share to conduct Task 3
					response = n_share.lookup(key, rings_processed_list);
					if response.found_flag == True then
						return response.key_info;
