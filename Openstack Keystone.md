<h1>Identity Service</h1>
<h1>Mục lục</h1>

<h4><a href="#tong_quan">1.	Giới thiệu tổng quan</a></h4>

<a href="#tq_khaiNiem">1.1. Các khái niệm</a>
<ul>
  <li><a href="#khaiNiem_project">1.1.1.	Project</a></li>
  <li><a href="#khaiNiem_domain">1.1.2.	Domain</a></li>
  <li><a href="#khaiNiem_user_group">1.1.3.	User và User Group (Actor)</a></li>
  <li><a href="#khaiNiem_role">1.1.4.	Role</a></li>
  <li><a href="#khaiNiem_token">1.1.5.	Token</a></li>
  <li><a href="#khaiNiem_catalog">1.1.6.	Catalog</a></li>
</ul>

<a href="#tq_identity">1.2.	Identity</a>
<ul>
  <li><a href="#iden_sql">1.2.1.	SQL</a></li>
  <li><a href="#iden_ldap">1.2.2.	LDAP</a></li>
  <li><a href="#iden_backends">1.2.3.	Multiple Backend</a></li>
  <li><a href="#iden_provider">1.2.4.	Identity Provider</a></li>
  <li><a href="#iden_usecase_backend">1.2.5.	Các trường hợp sử dụng các Identity backend</a></li>
</ul>

<a href="#tq_Authentication">1.3.	Authentication</a>
<ul>
  <li><a href="#authen_pass">1.3.1.	Password</a></li>
  <li><a href="#authen_token">1.3.2.	Token</a></li>
</ul>

<a href="#tq_author">1.4.	Access Management và Authorization</a></br>
<a href="#tq_format">1.5.	Các định dạng Token</a>
<ul>
  <li><a href="#format_uuid">1.5.1.	UUID	</a></li>
  <li><a href="#format_pki">1.5.2.	PKI – PKIZ</a></li>
  <li><a href="#format_fernet">1.5.3.	Fernet</a></li>
</ul>

<a href="#tq_activity">1.6.	Hoạt động của Keystone</a>
<h4><a href="#install_config">2.	Cài đặt và cấu hình Keystone</a></h4>
----

<h2><a name="tong_quan">1.	Giới thiệu tổng quan</a></h2>
&emsp;Keystone là dịch vụ xác thực của OpenStack – cung cấp quyền truy cập đảm bảo đã được kiểm soát đến tài nguyên của một cloud. Trong môi trường OpenStack, Keystone thực hiện nhiều chức năng quan trọng, như xác thực người dùng và xác định những tài liệu nào người dùng được quyền truy cập đến.</br>
&emsp;Để hiểu rõ cách Keystone cung cấp truy cập an toàn và được kiểm soát đến các tài nguyên, chúng ta cần xem xét các tính năng cơ bản của Keystone:</br>
<b>Indentity</b></br>
&emsp;Identity là định danh của người đang truy cập các tài nguyên cloud. Trong OpenStack keystone, identity thường đại diện cho một người dùng (user). Trong triển khai đơn giản nhất, identity của user có thể được lưu trữ trong database của Keystone. Nhưng trong môi trường sản xuất hay trong môi trường doanh nghiệp, một Identity Provider bên ngoài thường được sử dụng. Ví dụ như Manager Identity Federated Tivoli của IBM. Keystone có thể lấy ra thông tin định danh của người dùng từ các Provider Identity bên ngoài này.</br>
<b>Authentication</b></br>
&emsp;Authentication là quá trình của việc xác minh một thông tin định danh của người dùng. Một định danh người dùng đầu tiên sẽ được xác thực thông qua một password, sau đó các thông tin này sẽ được sử dụng để tạo ra một token để sử dụng các lần xác thực tiếp theo. Việc này sẽ làm giảm số lượng lần hiển thị và tiếp xúc với password (password được ẩn giấu và bảo vệ các nhiều càng tốt). Token cũng có giá trị trong một thời gian giới hạn (Việc này làm giảm khả năng sử dụng của nó trong trường hợp bị trộm). Trong OpenStack, Keystone là dịch vụ duy nhất có thể tạo ra token. </br>
&emsp;Hiện nay, Keystone sử dụng một dạng token được gọi là bearer token. Cụ thể, bất kỳ ai nắm quyền sở hữu token, cho dù là đúng hay là sai (vd: kẻ trộm,…) thì đều có khả năng sử dụng thẻ token để xác thực và truy cập đến các tài nguyên. Do đó, khi sử dung Keystone thì việc bảo vệ token là rất quan trọng.</br>
<b>Access Management (Authorication)</b></br>
&emsp;Access Management hay Authorication là quá trình xác định những tài nguyên nào một người dùng có thể truy cập.</br>
<b>các tính năng của Keystone</b></br>
&emsp;Trong khi Keystone tập trung hầu hết vào cung cấp các dịch vụ Identity, Authentication và Access Management, nó cũng hỗ trợ một số tính năng khác cho môi trường OpenStack:</br>
<ul>
<li>Xác thực đơn và cấp quyền cho các dịch vụ khác của OpenStack</li>
<li>Xử lý các hệ thống xác thực ngoài và cô lập tất cả các dịch vụ trong OpenStack</li>
<li>Keystone cung cấp các Project giúp các dịch vụ khác của OpenStack có thể sử dụng để phân biệt các tài nguyên (ví dụ như các máy chủ, các image,…).</li>
<li>Keystone cung cấp các Domain được sử dụng để định nghĩa name-space của các user, group và project để cho phép phân biệt giữa các người dùng</li>
<li>Cung cấp việc đăng ký các Roles được sử dụng cho việc cấp phép giữa Keystone và file policy của mỗi dịch vụ OpenStack</li>
<li>Cung cấp một catalog lưu trữ các dịch vụ OpenStack, endpoint và region, cho phép client có thể tìm kiếm các dịch vụ hoặc các endpoint mà họ cần.</li>
</ul>
<h3><a name="tq_khaiNiem">1.1.	Các khái niệm </a></h3></br>
Để có thể tìm hiểu Keystone, trước tiên cần phải hiểu các khái niệm được sử dụng trong Keystone.</br>
<h3><a name="khaiNiem_project">1.1.1.	 Project </a></h3></br>
&emsp;Trong Keystone, Project là khái niệm trừu tượng được sử dụng bởi các dịch vụ OpenStack để nhóm và phân biệt các tài nguyên (vd: các server, các image,…)</br>
&emsp;Trước đây, các Project Keystone sử dụng khái niệm “tenant”, nhưng do trực quan hơn nên đã đổi sang “Project”. Khái niệm này phù hợp hơn để nói về mục tiêu cơ bản nhất của Keystone là cung cấp các project và trình bày ai là được phép truy cập đến các project này.</br>
&emsp;Người dùng và nhóm người dùng nhận quyền truy cập đến một project sử dụng một khái niệm là Role Assignment (phân công). Một Role được gán cho một user hoặc user group trong một Project cho biết user hoặc user group có một số quyền truy cập đến các tài nguyên trong Project.</br>
&emsp;Ví dụ:</br>
-	Một người có thể thuộc nhiều project và có các quyền khác nhau.</br>
-	“Alice” có quyền admin trong project “abc” nhưng chỉ có quyền user trong project “xyz”</br>
<h3><a name="khaiNiem_domain">1.1.2.	 Domain</a></h3></br>
&emsp; Trong giao đoạn đầu của OpenStack, không có cơ chế để hạn chế tầm nhìn của các project trong các tổ chức người dùng khác nhau. Điều này dẫn đến nhưng va chạm về tên Project của các tổ chức khác nhau. Tên user cũng có tầm nhìn tổng thể nên cũng có thể dẫn đến những va chạm không mong muốn khi 2 tổ chức đều có chung một tên user. Keystone đã đưa ra khái niệm là Domain.</br>
&emsp; Domain cung cấp khả năng để cô lập tầm nhìn của một tập các Project và user (và group) trong một tổ chức cụ thể. Một Domain được định nghĩa như là một tập hợp bao gồm các user, các group và các project.</br>
<h3><a name="khaiNiem_user_group">1.1.3.	 User và User Group (Actor)</a></h3></br>
&emsp;Trong Keystone, user và user group là các thực thể được cấp quyền truy cập đến các tài nguyên trong các project. Group là một tập hợp của nhiều user. User là đại diện cho một người, hay một dịch vụ sử dụng các dịch vụ OpenStack và được chứa trong một Domain cụ thể.</br>
&emsp;Mối quan hệ của Domain, project, user và group được thể hiện trong hình sau:</br>
<img src="img_KeyStone/relationship.png"/><br><br>
&emsp;Các user, group và project luôn luôn nằm trong phạm vi của Domain. Điều này dẫn đến, một tên của user hay groups hay project có thể được sử dụng trên các Domain khác nhau. Ví dụ như user “Alice” hoặc groups “Adminstrator” đều có ở cả IBM và Acme Inc.</br>
<h3><a name="khaiNiem_role">1.1.4.	 Role</a></h3></br>
&emsp;Role được sử dụng trong Keystone để xác định những quyền mà môt user hay group có thể thực hiện với một tài nguyên OpenStack.</br>
&emsp;Ví dụ, Role admin được gán cho user “Alice” đối với project “Development”. Khi đó, user “Alice” sẽ thực hiện được các hành động của quyền admin đối với các tài nguyên của project “Development”</br>
<h3><a name="khaiNiem_token">1.1.5.	 Token</a></h3></br>
&emsp;Khi người dùng gọi đến bất kỳ một API OpenStack thì họ phải chứng minh được rằng họ là ai và họ phải được cho phép để gọi đến các API. Thực hiện việc này bằng cách truyền một token OpenStack vào trong lời gọi API. Keystone là dịch vụ duy nhất chịu trách nhiệm tạo ra token. Người dùng nhận một token khi xác thực thành công với Keystone. Token chứa quyền của một user trong cloud. Nó bao gồm một ID và một payload. ID đảm bảo tính duy nhất của token trên mỗi cloud và payload chứa dữ liệu về người dùng.</br>
&emsp;Ví dụ: một token sẽ chứa nhiều trường thể hiện các thuộc tính Identity và Authorization về một user trong một project.</br>
 <img src="img_KeyStone/exTokenV3.png"/><br><br>
&emsp;Token trên chứa những thông tin sau: Khi nào token được tạo ra; Khi nào token hết hạn, user nào đã được xác thực- do đó, được phép sử dụng token này; xác định project nào token này có giá trị; Và cuối cùng là Catalog chứa các dịch vụ trong OpenStack.</br>
<h3><a name="khaiNiem_catalog">1.1.6.	 Catalog</a></h3></br>
&emsp;Catalog là danh mục chứa các URL và các endpoint của các dịch vụ OpenStack khác nhau. Nó rất cần thiết cho OpenStack, thiếu catalog thì các user, và các ứng dụng sẽ không biết địa chỉ để gửi request tạo một máy ảo hoặc là lưu trữ một đối tượng.</br>
&emsp;Catalog được chứa các endpoint và mỗi endpoints chứa một admin URL, enternal URL và public URL.</br>
&emsp;Ví dụ về một Catalog chỉ chứa hai dịch vụ Object Storage và Identity</br>
  <img src="img_KeyStone/exCatolog.png"/><br><br>
&emsp;Trên đây là các khái niệm trong OpenStack, tiếp theo sẽ đi vào chi tiết các tính năng của Keystone.</br>
<h3><a name="tq_identity">1.2.	Identity</a></h3></br>
&emsp;Dịch vụ Identity trong Keystone cung cấp các Actor (user hay group). Identity trong Openstack có thể đến từ các vị trí khác nhau như SQL, LDAP hay Federated Identity Provider.</br>
<h3><a name="iden_sql">1.2.1. SQL</a></h3></br>
&emsp;Keystone hỗ trợ các database để lưu trữ các actor như MySQL, PostgreSQL hay DB2. Keystone sẽ lưu trữ các thông tin của người dùng như tên, password và mô tả. </br>
&emsp;Về cơ bản, Keystone hoạt động như một Identity Provider như chắc chắn rằng sẽ không phải là một giải pháp tốt ưu trong một số trường hợp như trong môi trường doanh nghiệp.</br>
&emsp;Các ưu, nhược điểm khi sử dụng SQL:</br>
<ul>
    <li>Ưu điểm:
        <ul>
            <li>Dễ dàng cài đặt</li>
            <li>Quản lý user và group thông qua các API OpenStack</li>
        </ul>
    </li>
    <li>Nhược điểm:
        <ul>
            <li>Keystone không phải là một Identity Provider</li>
            <li>Hỗ trợ password yếu: Không luân chuyển password và không khôi phục password</li>
            <li>Hầu hết các doanh nghiệp sử dụng LDAP</li>
        </ul>
    </li>
</ul>
<h3><a name="iden_ldap">1.2.2.	 LDAP</a></h3></br>	
&emsp;Keystone cũng có một tùy chọn để lưu trữ và lấy dữ liệu actor trong Lightweight Directory Access Protocol (LDAP). Keystone sẽ truy cập đến LDAP như các ứng dụng khác sử dụng LDAP (như Email, Web application,…). Thiết lập kết nối đến LDAP trong file cấu hình Keystone. </br>	
&emsp;LDAP thường chỉ thực hiện quyền đọc, chẳng hạn như tìm kiếm user (thông qua search) và xác thực (thông qua bind).</br>	
&emsp;Ưu, nhược điểm khi sử dụng LDAP:</br>	
<ul>
    <li>Ưu điểm:
        <ul>
            <li>Không duy trì bản sao của tài khoản user</li>
            <li>Keystone hoạt động như Identity Provider (nhà cung cấp nhận dạng).</li>
        </ul>
    </li>
    <li>Nhược điểm:
        <ul>
            <li>Các tài khoản của các dịch vụ khác trong OpenStack cũng cần được lưu trữ ở đâu đó, và admin LDAP không muốn lưu các tài khoản này trong LDAP</li>
            <li>Keystone vẫn thấy được password của user, khi password nằm trong một yêu cầu xác thực. Keystone đơn giản chỉ chuyển các yêu cầu, vì vậy không cần thiết phải biết password của user</li>
        </ul>
    </li>
</ul>
<h3><a name="iden_backends">1.2.3. Multiple Backend</a></h3></br>
&emsp;Keystone hỗ trợ sử dụng multiple backend, nghĩa là mỗi domain có thể sử dụng một backend</br>
&emsp;Ví dụ: Trong ví dụ sau, Identity sử dụng LDAP cho Domain A và Domain B, một SQL để lưu trữ các service account và các Assignment</br>
<img src="img_KeyStone/multiple_backend.png"/><br><br>
&emsp;Ưu, nhược điểm khi sử dụng multiple backend:</br>
<ul>
    <li>Ưu điểm:
        <ul>
            <li>Hỗ trợ đồng thời nhiều backend</li>
            <li>Có thể sử dụng lại các LDAP đã có</li>
        </ul>
    </li>
    <li>Nhược điểm:
        <ul>
            <li>Cài đặt khá phức tạp</li>
            <li>Xác thực người dùng bắt buộc phải nằm trong phạm vi Domain</li>
        </ul>
    </li>
</ul>
<h3><a name="iden_provider">1.2.4. Identity Provider</a></h3></br>
&emsp;Từ bản IceHouse, keystone hỗ trợ federate authentication thông qua Apache Module với nhiều Identity Provider. User không lưu trữ ở keystone. Identity Provider là các nguồn lưu trữ thông tin về Identity được lưu bởi các loại backend như Ldap, AD, MongoDB hoặc Social Login (FB, Twitter).</br>
&emsp;Ưu, nhược điểm khi sử dụng identity provider:</br>
<ul>
    <li>Ưu điểm:
        <ul>
            <li>Có thể tích hợp với các hệ thống có sẵn để xác thực người dùng và lấy các thông tin người dùng sẵn có</li>
            <li>Có sự tách biệt giữa keystone và xử lý thông tin định danh.</li>
            <li>Mở ra khả năng single sign-on và hibrid cloud.</li>
            <li>Keystone không thể xem được password của user (an toàn).</li>
            <li>Identity Provider xử lý tất cả các công việc xác thực, vì vậy, việc sử dụng kiểu xác thực nào (password, cert, two-factor,…) không liên quan đến keystone</li>
        </ul>
    </li>
    <li>Nhược điểm:
        <ul>
            <li>Phải cài đặt phức tạp</li>
        </ul>
    </li>
</ul>
<h3><a name="iden_usecase_backend">1.2.5.	Các trường hợp sử dụng các Identity backend</a></h3></br>
&emsp;Tùy từng trường hợp cụ thể, có thể lựa chọn sử dụng các identity backend phù hợp.
<ul>
    <li>SQL:
        <ul>
            <li>Thử nghiệm hoặc phát triển OpenStack</li>
            <li>Với một tập hợp ít user</li>
            <li>Có các service account.</li>
        </ul>
    </li>
    <li>LDAP:
        <ul>
            <li>Trong môi trường doanh nghiệp</li>
            <li>Chỉ sử dụng nếu có khả năng tạo service account cần thiết trong LDAP</li>
        </ul>
    </li>
     <li>Multiple Backend:
        <ul>
            <li>Hầu hết được sử dụng trong các doanh nghiệp</li>
            <li>Sử dụng trong trường hợp LDAP không cho phép service account</li>
        </ul>
    </li>
     <li>Identity Provider:
        <ul>
            <li>Muốn tận dụng cơ chế mới Federated Identity</li>
            <li>Keystone không thể truy cập được LDAP</li>
        </ul>
    </li>
</ul>
<h3><a name="tq_Authentication">1.3.	Authentication </a></h3></br>
&emsp;Keystone sử dụng hai cách để xác thực là cung cấp password hoặc sử dụng một token.</br>
<h3><a name="authen_pass">1.3.1.	Password</a></h3></br>
&emsp;Cách phổ biến là các user hay service có thể xác thực là cung cấp một password. Ví dụ dưới đây là payload request đến keystone. Nó thể hiện tất cả thông tin cần thiết cho việc xác thực bao gồm thông tin user và scope của user đó</br>
  <img src="img_KeyStone/password.png"/><br><br>
&emsp;Phần scope là không bắt buộc,nhưng nó thường xuyên được dùng. Bởi vì, thiếu scope user sẽ không lấy được một catalog các service. Scope được sử dụng để chỉ ra những project mà người dùng muốn làm việc. Nếu người dùng không có role trong project này, thì yêu cầu bị hủy bỏ. Cũng giống như phần user, phần scope cũng phải có đầy đủ thông tin về project mà user muốn tìm. Việc này để tránh trương hợp va chạm tên Project.</br>
<h3><a name="authen_token">1.3.2.	Token</a></h3></br>
&emsp;Tương tự như password, user yêu cầu một token mới bằng cách cung cấp token hiện tại. Dưới đây là một ví dụ về một request token mới</br> 
 <img src="img_KeyStone/exToken.png"/><br><br>
 <h3><a name="tq_author">1.4.	Access Management và Authorization </a></h3></br>
&emsp;KeyStone tạo ra một chính sách Role-Based Access Control (RBAC), thực thi tại mỗi public API endpoint. Các chính sách được lưu trong file policy.json, nó bao gồm mục tiêu và quy tắc. Mỗi luật bắt đầu với " identity:" </br>
<img src="img_KeyStone/password_diagram.png"/><br><br>
<h3><a name="tq_format">1.5.	Các định dạng Token </a></h3></br>
<h3><a name="format_uuid">1.5.1.	UUID </a></h3></br>
&emsp;Trong những ngày đầu tiên, định dạng token của Keystone là UUID format. UUID format đơn giản chỉ lại một chuỗi 32 ký tự được sinh ngẫu nhiên. Token này được cung cấp và được xác thực bởi dịch vụ identity. </br>
&emsp;Lợi ích của Token format này là token nhỏ (ngắn) và dễ sử dụng, đủ đơn giản để có thể thêm vào trong các lệnh cURL. </br>
&emsp;Tuy nhiên nhược điểm của nó là không mang đủ những thông tin để có thể xác thực một cách trực tiếp các dữ liệu và request. Các dịch vụ của openstack cứ liên tục phải gửi lại token về phía keystone server để xác thực các request đến các service đó. Dẫn đến bất kì hành động nào trong Openstack đều phải thông qua keystone server.</br>
<b>UUID Token Generation Workflow</b></br>
<img src="img_KeyStone/uuid-token_generation.png"/><br><br>
&emsp;Khi user gửi yều cầu tạo token đến Keystone với các thông tin Username, password và Project Name, Keystone sẽ thực hiện các bước sau để tạo ra UUID Token</br>
<ol>
    <li>Xác nhận user và lấy user ID</li>
    <li>Xác nhận project và lấy project ID và domain ID</li>
    <li>Lấy các Role của user trên project hoặc domain. Trả về failure, nếu user không có quyền nào.</li>
    <li>Lấy các service và endpoint của tất các các service trong OpenStack</li>
    <li>Đóng gói các thông tin Identity, Resource, Assignment và Catalog vào payload token và tạo Token ID</li>
&emsp;Lưu giữ các thông tin Token ID, Expiration, Valid, User ID, Extra vào backend.</br>
</ol>
<b>UUID Token Validation Workflow</b></br>
<img src="img_KeyStone/uuid-token_validation.png"/><br><br>
&emsp;Quá trình xác minh UUID Token như sau:</br>
<ol>
    <li>Sử dụng câu lệnh “GET v3/auth/tokens” để gửi token đến Keystone </li>
    <li>Lấy payload Token từ backend và kiểm tra các giá trị có đúng hay không? Nếu sai “Token Not Found”, Nếu đúng thực hiện bước tiếp theo.</li>
    <li>Phân tích token và lấy metadata của token đó (bao gồm User ID, Project ID, Audit  ID, và Token Expiry).</li>
    <li>Kiểm tra xem token đã hết hạn hay chưa (sử dung thời gian hiện tại được tính toàn trong UTC)? Nếu đã quá hạn (thời gian hiện tại lớn hơn thời gian hết hạn) thì báo “Token Not Found”, nếu đúng thực hiện bước tiếp theo.</li>
    <li>Kiểm tra xem Token đã bị thu hồi hay chưa? Nếu đã bị thu hồi, thì trả về “Token Not Found”. Nếu đúng, trả về “HTTP/1.1 200 OK” (có nghĩa là token sử dụng được)</li>
</ol>
<b>UUID Token Revocation Workflow</b></br> 
<img src="img_KeyStone/uuid-token_revocation.png"/><br><br>
&emsp;Quá trình thu hồi một token thực hiện như sau:</br>
<ol>
    <li>Gửi request “DELETE v3/auth/tokens” đến Keystone để yêu cầu thu hồi token. Trước khi thu hồi token, thì phải thực hiện xác minh token (thực hiện các bước như trên)</li>
    <li>Kiểm tra xem có Audit ID không? Nếu không chuyển sang bước 3. Nếu có thì chuyển sang bước 6</li>
    <li>Token được thu hồi khi hết hạn.</li>
    <li>Tạo event thu hồi với các thông tin sau: User ID, Project ID, Revoke At, Issued Before và Token Expiry</li>
    <li>Chuyển đến bước 8.</li>
    <li>Thu hồi bởi Audit ID.</li>
    <li>Tạo event thu hồi với các thông tin: Audit ID, Revoke At, Issued Before</li>
    <li>Lọc các event revoke đang tồn tại dựa trên revoke at</li>
    <li>Cài đặt giá trị false cho token kvs</li>
</ol>
<h3><a name="format_pki">1.5.2.	PKI – PKIZ</a></h3></br>
&emsp;Fomat token thứ 2 mà Keystone hỗ trợ là PKI format. Trong format này, token sẽ chứa toàn bộ thông tin xác minh chẳng hạn như: Khi nào token được cung cấp, khi nào token hết hạn, định danh người dùng, project, domain, role và service catalog. Tất cả những thông tin này được đóng gói trong một payload.</br>
&emsp;Muốn gửi token qua HTTP, JSON token payload phải được mã hóa base64 với 1 số chỉnh sửa nhỏ. Cụ thể, Format=CMS+[zlib] + base64. Ban đầu JSON payload phải được ký sử dụng một khóa bất đối xứng(private key), sau đó được đóng gói trong CMS (cryptographic message syntax - cú pháp thông điệp mật mã). Với PKIz format, sau khi đóng dấu, payload được nén lại sử dụng trình nén zlib. Tiếp đó PKI token được mã hóa base64 và tạo ra một URL an toàn để gửi token đi.</br>
&emsp;Các OpenStack services cache lại token này để đưa ra quyết định ủy quyền mà không phải liên hệ lại keystone mỗi lần có yêu cầu ủy quyền dịch vụ cho user.</br>
&emsp;Kích thước của 1 token cơ bản với single endpoint trong catalog lên tới 1700 bytes. Với các hệ thống triển khai lớn nhiều endpoint và dịch vụ, kích thước của PKI token có thể vượt quá kích thước giới hạn cho phép của HTTP header trên hầu hết các webserver(8KB). Thực tế khi sử dụng chuẩn token PKIz đã nén lại nhưng kích thước giảm không đáng kể (khoảng 10%).</br>
&emsp;PKI và PKIz tokens tuy rằng có thể cached nhưng chúng có nhiều hạn chế</br>
<ul>
    <li>Khó cấu hình để sử dụng</li>
    <li>Kích thước quá lớn làm giảm hiệu suất web</li>
    <li>Khó khăn khi sử dụng trong cURL command.</li>
    <li>Keystone phải lưu các token với rất nhiều thông tin trong backend database với nhiều mục đích, chẳng hạn như tạo danh sách các token đã bị thu hồi. Hệ quả là người dùng phải lo về việc phải flush Keystone token database định kì tránh ảnh hưởng hiệu suất.</li>
</ul>
<b>Token PKI/PKIZ Generation Workflow</b></br>
<img src="img_KeyStone/pki-token_generation.png"/><br><br>
<ol>
    <li>User gửi request đến Keystone.</li>
    <li>Xác minh identity, resource (project và domain), assignment</li>
    <li>Tạo JSON Token Payload</li>
    <li>“Ký" lên JSON payload với Signing Key và Signing Certificate , sau đó được đóng gói lại dưới định dang CMS (cryptographic message syntax - cú pháp thông điệp mật mã)</li>
    <li>Bước tiếp theo, nếu muốn đóng gói token định dạng PKI thì convert payload sang UTF-8, convert token sang một URL định dạng an toàn. Nếu muốn token đóng gói dưới định dang PKIz, thì phải nén token sử dụng zlib, tiến hành mã hóa base64 token tạo ra URL an toàn, convert sang UTF-8 và chèn thêm tiếp đầu ngữ "PKIZ"</li>
    <li>Lưu vào Backend</li>
</ol>
<b>Token PKI/PKIZ Validation Workflow</b></br>
&emsp;Tương tự như UUID, chỉ khác ở bước gửi yêu cầu, Keystone sẽ băm lại PKI Token với thuật toán băm đã cấu hình trước. Các bước tiếp theo tương tự UUID Token</br>
<b>Token PKI/PKIZ Revocation Workflow</b></br>
&emsp;Tương tự như UUID.</br>
<h3><a name="format_fernet">1.5.3.	Fernet</a></h3></br>
&emsp;Format mới nhất mà Keystone hỗ trợ là Fernet format. Fernet token chứa tối đa 255 ký tự, lớn hơn UUID và bé hơn rất nhiều so với PKI. Token này chỉ chứa đầy đủ thông tin của một token để không cần phải lưu trong database.</br>
&emsp;Fernet token cũng được “Ký” sử dụng khóa đối xứng để ngăn chặn giả mạo. Và các khóa này cần phải được phân bổ đến các OpenStack region khác nhau.</br>
<b>Fernet key</b></br>
&emsp;Một key reponsitory được yêu cầu bởi Keystone để lưu trữ các Key dùng để mã hóa và giải mã các thông tin của token. Mỗi Key trong reponsitory có 3 trạng thái:</br>
<ul>
    <li>Primary key:  Một key reponsitory chỉ có duy nhất một primary key. Sử dụng để mã hóa và giải mã token. Key này luôn đặt tên là index cao nhất trong key reponsitory</li>
    <li>Secondary key: là một thời điểm của primary key, nhưng nó đã bị giảm vị trí của một primary key khác. Key này chỉ được dùng để giải mã token. Keystone sữ dụng để giải mã các token được mã hóa bởi các primary key cũ.</li>
    <li>Staged key: Là một Key đặc biệt, có một số điểm tương đồng với Secondary key. Một key reponsitory chỉ có duy nhất một primary key. Nó giống với Secondary key là chỉ có khả năng giải mã token. Staged key sẽ là primary key trong lên chuyển khóa tiếp theo. Nó được đặt tên là index 0 trong key reponsitory.</li>
</ul>
&emsp;Fernet key có một chu kỳ, được bắt đầu từ Staged key, sau đó được nâng lên Primary key và cuối cùng giảm xuống Secondary key. Fernet key được lưu trữ trong file “/etc/keystone/fernet-keys”</br>
<b>Fernet key rotation</b></br>
<img src="img_KeyStone/fernet_key_rotation.png"/><br><br>
&emsp;Quá trình rotate key cụ thể như sau:</br>
<ul>
    <li>Ban đầu: Key reponsitory sẽ có Staged key và primary key. Không có secondary key.</li>
    <li>Rotate lần đầu tiên: Staged key sẽ trở thành primary key. Primary key chuyển thành secondary key. Một Staged key mới được tạo ra</li>
    <li>Rotate lầu thứ 2: Staged key Staged key sẽ trở thành primary key. Primary key chuyển thành secondary key thứ 2 trong key reponsitory. Một Staged key mới được tạo ra.</li>
    <li>Tương tự như vậy cho các lần tiếp theo.</li>
</ul>
&emsp;Vậy thì key reponsitory sẽ bị quá tải khi số lượng secondary key tăng lên? Không, key reponsitory sẽ có số lượng key cố định, khi vượt mức thì nó sẽ xóa secondary key xuất hiện sớm nhất.</br>
<b>Token Generation Workflow</b></br>
<img src="img_KeyStone/fernet-token_generation.png"/><br><br>
&emsp;Sau khi nhận được yêu cầu tạo token từ user Keystone sẽ thực hiện quá trình tạo token:</br>
&emsp;Như hình trên, token bao gồm các trường sau:</br>
<ol>
    <li>Fernet token version: cho biết phiên bản của định dạng token </li>
    <li>Current Timestamp : nhãn thời gian hiện tại, chỉ ra thời điểm token được tạo ra</li>
    <li>IV – Initialation : sử dụng để mã hóa và giãi mã </li>
    <li>Cipher text: là sự kết hợp của token payload (với các thông tin như version, user ID, Methods, Project ID,…) và Padding message. Sau đó, encrypt sử dụng encryption key được cung cấp</li>
    <li>HMAC: là tổng hợp tất cả các trường trên và “ký” sử dụng signing key mà user cung cấp </li>
</ol>
<b>Token Validation Workflow</b></br>
<img src="img_KeyStone/fernet-token_validation.png"/><br><br>
&emsp;Quá trình validation token cụ thể như sau:</br>
<ol>
    <li>User gửi yêu cầu validation đến keystone với phương thức: GET v3/auth/tokens</li>
    <li>Khôi phục lại Padding, trả về token với padding chính xác</li>
    <li>Giải mã token sử dụng fernet key để lấy token payload</li>
    <li>Xác định version từ token payload. Version cố định bởi keystone (unscope payload: 0; domain scope payload: 1; project scope payload: 2).</li>
    <li>Phân tích payload để xác minh các trường. ví dụ, trong phạm vi một project có các trường: user ID, project ID, Methods, Audit ID, expiry</li>
    <li>Kiểm tra xem token đã hết hạn chưa? Nếu đã hết hạn (thời gian hiện tại lớn hơn thời gian hết hạn), thì trả về “Token Not Found”. Nếu chưa hết hạn, tiếp tục bước sau.</li>
    <li>Kiểm tra token đã bị thu hồi hay chưa? Nếu đã bị thu hồi, trả về “Token not found”. Nếu đúng, trả về “HTTP/1.1 200 OK” token đã được validation</li>
</ol>
<b>Token Validation Workflow</b></br>
&emsp;Tương tự UUID và PKI/ PKIZ </br>
<b>Multiple Data Center</b></br>
<img src="img_KeyStone/fernet-multiple_data_center.jpg"/><br><br>
&emsp;Giả sử triển khai hệ thống cloud với keystone ở cả hai bên US-WEST và US-EAST. Cả hai bên này đều có LDAP và Database luôn luôn được đồng bộ. Hoạt động của keystone khi user muốn thực hiện tạo một máy ảo VM như sau:</br>
<ul>
    <li>User sử dụng Nova ở US-WEST:
        <ul>
            <li>Đầu tiên, user sẽ yêu cầu token đến keystone. Keystone sẽ tạo ra một fernet token cho user (mô tả trong Token Generation Workflow). Trả lại user fernet token</li>
            <li>User gửi yều cầu tạo máy ảo đến Nova và fernet token của mình. </li>
            <li>Nova thực hiện validation token bằng cách gửi yêu cầu validation token đến cho Keystone. Keystone thực hiện validation token (các bước cụ thể mô tả ở trên). Keystone gửi kết quả lại cho Nova </li>
            <li>Nếu xác minh đúng, Nova thực hiện yêu cầu tạo máy ảo và gửi phản hồi cho user</li>
        </ul>
    </li>
    <li>
        <ul>
            <li>User gửi yều cầu tạo máy ảo đến Nova và fernet token của mình (đã được cấp bởi keystone bên US-WEST). </li>
            <li>Nova thực hiện validation token bằng cách gửi yêu cầu validation token đến cho Keystone. Keystone thực hiện validation token (các bước cụ thể mô tả ở trên). Keystone gửi kết quả lại cho Nova </li>
            <li>Nếu xác minh đúng, Nova thực hiện yêu cầu tạo máy ảo và gửi phản hồi cho user</li>
        </ul>
    </li>
</ul>
<h3><a name="tq_activity">1.6.	Hoạt động của Keystone</a></h3></br>
&emsp;Biểu đồ tuẩn tự dưới đây sẽ mô tả quá trình hoạt động của Keystone trong ví dụ cụ thể là tạo một máy ảo VM.</br>
<img src="img_KeyStone/keystone_process_flow.png"/><br><br>
&emsp;Cụ thể các bước như sau :</br>
<ol>
    <li>User gửi thông tin username và password đến keystone.</li>
    <li>Keystone kiểm tra thông tin. Nếu thông tin đúng (xác thực thành công), thì gửi một token lại cho user</li>
    <li>User gửi token và yêu cầu đến Nova.</li>
    <li>Nova gửi token (của user gửi đến) cho Keystone để yêu cầu xác thực có đúng không? Có những quyền hạn gì? Và Keystone sẽ đáp trả lại cho Nova</li>
    <li>Nếu đúng, Nova token (của nova) và yêu cầu image đến cho Glance.</li>
    <li>Glance gửi token (của Nova gửi đến) cho Keystone để xác thực và kiểm tra quyền đối với file image? Keystone đáp trả lại cho Glance.</li>
    <li>Glance gửi image lại cho Nova.</li>
    <li>Nova gửi token và yêu cầu cung cấp mạng đến Neutron.</li>
    <li>Neutron gửi token (của Nova gửi đến) cho Keystone để xác thực và kiểm tra quyền của Nova. Keystone đáp trả lại cho Neutron.</li>
    <li>Neutron phản hồi lại cho Nova.</li>
    <li>Nova thực hiện tiếp quá trình và gửi phản hồi lại cho user </li>
</ol>
<h3><a name="install_config">Cấu hình và cài đặt Keystone</a></h3>
- Trước tiên, cần phải tạo ra một database cho dịch vụ keystone bằng các câu lệnh sau:
```sh
mysql -u root –pWelcome123
CREATE DATABASE keystone;

GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
IDENTIFIED BY 'Welcome123';

GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
IDENTIFIED BY 'Welcome123';
flush privileges;
exit;
```
- Cấu hình không cho dịch vụ keystone tự động khởi động:
```sh
echo "manual" > /etc/init/keystone.override
```
- Chạy lệnh sau để cài đặt các gói của keystone:
```sh
apt-get install keystone apache2 libapache2-mod-wsgi
```
- Cấu hinh file `/etc/keystone/keystone.conf` với các yêu cầu sau:
 - Trong phần `[default]`, định nghĩa giá trị của thẻ quản trị ban đầu:
  ```sh
  [DEFAULT]
  ...
  admin_token = Welcome123
  ```
 - Trong phần database, cấu hình truy cập đến database:
  ```sh
  [database]
  ...
  connection = mysql+pymysql://keystone:Welcome123@controller/keystone
  ```
  - Trong phần [token], cấu hình nhà cung cấp thẻ Fernet:
  ```sh  
  [token]
  ...
  provider = fernet
  ```
- Đồng bộ database dịch vụ xác thực:
  ```sh
  su -s /bin/sh -c "keystone-manage db_sync" keystone
  ```
- Thiết lập Fernet key:
  ```sh
  keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
  ```
<h3>cấu hình máy chủ Apache</h3>
- Chỉnh sửa file `/etc/apache2/apache2.conf` và cấu hình tùy chọn Servername để ánh xạ đến node controller:
```sh
ServerName controller
```
- Tạo file `/etc/apache2/sites-available/wsgi-keystone.conf` với nội dung sau:
```sh
Listen 5000
Listen 35357

<VirtualHost *:5000>
    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /usr/bin/keystone-wsgi-public
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/apache2/keystone.log
    CustomLog /var/log/apache2/keystone_access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>

<VirtualHost *:35357>
    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
WSGIProcessGroup keystone-admin
    WSGIScriptAlias / /usr/bin/keystone-wsgi-admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/apache2/keystone.log
    CustomLog /var/log/apache2/keystone_access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>
```
- Cấp phép các máy ảo dịch vụ xác thực:
```sh
ln -s /etc/apache2/sites-available/wsgi-keystone.conf /etc/apache2/sites-enabled
```
- Khởi động lại apache:
```sh
service apache2 restart
```
- Xóa database mặc định của keystone:
```sh
rm -f /var/lib/keystone/keystone.db
```
<h3>Tạo endpoint và các service cho keystone</h3>
- Vì ban đầu, database của Keystone không chứa thông tin xác thực và catalog sevices nên để tạo được các endpoint và các service thì phải có một token để cho phép thực hiện bước này. 
- Truyền vào các biến môi trường để khởi tạo service và các endpoint indentity:
```sh
export OS_TOKEN=ADMIN_TOKEN
```
ADMIN_TOKEN ở đây là giá trị đã được khai báo trong file cấu hình ở bước trước.
- Khai báo URL endpoint và version API identity:
```sh
export OS_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
```
- Keystone quản lý một catalog các dịch vụ trong môi trường OpenStack. Các dịch vụ sử dụng catalog này để xác định các dịch vụ khác đang có trong môi trường.
- Tạo service cho dịch vụ identity:
```sh
openstack service create \
  --name keystone --description "OpenStack Identity" identity
```
- Keystone cũng quản lý một danh mục các endpoint API được kết nối với các dịch vụ trong môi trường OpenStack . Các dịch vụ sử dụng catalog này để xác định các giao tiếp với các dịch vụ khác trong môi trường OpenStack.
- Tạo các endpoints API: 
 - Public API endpoint:
 ```sh
 openstack endpoint create --region RegionOne \
 identity public http://controller:5000/v3
 ```
 - Internal API endpoint:
 ```sh
 openstack endpoint create --region RegionOne \
 identity internal http://controller:5000/v3
 ```
 - Admin API endpoint:
 ```sh
 openstack endpoint create --region RegionOne \
  identity public http://controller:5000/v3
 ```
<h3>Tạo domain, user, project và role</h3>
- Tạo domain `default`:
```sh
openstack domain create --description "Default Domain" default
```
- Tạo project `admin`:
```sh
openstack project create --domain default \
  --description "Admin Project" admin
```	
- Tạo user `admin`:
```sh
openstack user create --domain default \
  --password-prompt admin
```
- Và tạo role `admin`:
```sh
openstack role create admin
```	
- Gán role admin cho user `admin` đối với project `admin`:
```sh
openstack role add --project admin --user admin admin
```

<h3>Kiểm tra hoạt động</h3>
- Vì lý do bảo mật, vô hiệu hóa cơ chế thẻ token tạm thời bằng cách chỉnh sửa trong file `/etc/keystone/keystone-paste.ini`, xóa các dòng `admin_token_auth` từ các phần `[pipeline:public_api]`,`[pipeline:admin_api]` và `[pipeline:api_v3]`
- Gỡ bỏ các biến môi trường đã thiết lập trong quá trình tạo service và endpoint cho dịch vụ Identity.
- Kiểm tra hoạt động bằng cách yêu cầu token cho user “admin” đã tạo ở trên:
```sh
  openstack --os-auth-url http://controller:35357/v3 \
   --os-project-domain-name default --os-user-domain-name default \
   --os-project-name admin --os-username admin token issue
  Password:
```
  Nhập password vào và hiển thị kết quả như sau:
```sh
+------------+-----------------------------------------------------------------+
| Field      | Value                                                           |
+------------+-----------------------------------------------------------------+
| expires    | 2016-02-12T20:14:07.056119Z                                     |
| id         | gAAAAABWvi7_B8kKQD9wdXac8MoZiQldmjEO643d-e_j-XXq9AmIegIbA7UHGPv |
|            | atnN21qtOMjCFWX7BReJEQnVOAj3nclRQgAYRsfSU_MrsuWb4EDtnjU7HEpoBb4 |
|            | o6ozsA_NmFWEpLeKy0uNn_WeKbAhYygrsmQGA49dclHVnz-OMVLiyM9ws       |
| project_id | 343d245e850143a096806dfaefa9afdc                                |
| user_id    | ac3377633149401296f6c0d92d79dc16                                |
+------------+-----------------------------------------------------------------+
```
- Ngoài ra, để tăng hiệu quả của hoạt động ở phía client khi thực hiện yêu cầu token thì OpenStack hỗ trợ một kịch bản môi trường client
- Tạo kịch bản môi trường client cho user “admin” bằng cách tạo file `admin-openrc` với nội dung sau:
```sh
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```
- Sau khi tạo xong, thực hiện yêu cầu token đối với user “admin” sẽ thực hiện các bước sau:
- Chạy scripts vừa tạo :
```sh
. admin-openrc
```
- Yêu cầu token và thu được kết quả sau:
```sh
openstack token issue
+------------+-----------------------------------------------------------------+
| Field      | Value                                                           |
+------------+-----------------------------------------------------------------+
| expires    | 2016-02-12T20:44:35.659723Z                                     |
| id         | gAAAAABWvjYj-Zjfg8WXFaQnUd1DMYTBVrKw4h3fIagi5NoEmh21U72SrRv2trl |
|            | JWFYhLi2_uPR31Igf6A8mH2Rw9kv_bxNo1jbLNPLGzW_u5FC7InFqx0yYtTwa1e |
|            | eq2b0f6-18KZyQhs7F3teAta143kJEWuNEYET-y7u29y0be1_64KYkM7E       |
| project_id | 343d245e850143a096806dfaefa9afdc                                |
| user_id    | ac3377633149401296f6c0d92d79dc16                                |
+------------+-----------------------------------------------------------------+
```
