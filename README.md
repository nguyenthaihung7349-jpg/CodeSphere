# Project Overview
Dự án này là một nền tảng mạng xã hội chia sẻ bài viết (social blogging platform) được xây dựng trên ASP.NET Core MVC (.NET 9), với kiến trúc module hóa theo Areas, hỗ trợ chat riêng tư thời gian thực, thông báo người dùng, tác vụ nền (background jobs), machine learning để hỗ trợ kiểm duyệt nội dung và sinh nội dung tự động bằng AI thông qua tầng dịch vụ AI chuyên biệt.
Dự án kết hợp MVC truyền thống, Razor Pages (cho Identity) và Server-Side Blazor, với phân tách rõ ràng giữa Controllers, Repositories, Services, ML Models, AI Services và SignalR Hubs.

# Key Features
User Accounts & Roles
ASP.NET Core Identity với ApplicationUser / ApplicationRole tùy biến
Phân quyền theo vai trò (Administrator, Editor, v.v.)
Đăng nhập mạng xã hội qua Facebook và Google
Blogging & Content Management
Bài viết, chuyên mục (categories), thẻ (tags), bình luận, lượt thích, bài viết yêu thích, bài chờ duyệt/bị chặn
Soạn thảo nội dung rich text với TinyMCE
Hỗ trợ Markdown → HTML cho nội dung AI thông qua Markdig
Quản lý categories/tags cho biên tập viên và quản trị viên
Private Chat & Stickers
Chat 1-1 thời gian thực với SignalR (PrivateChatHub)
Chủ đề chat (chat themes), emoji, sticker, sticker yêu thích, quick replies, hình ảnh trong chat
Gói sticker/emoji được quản lý trong khu vực Administration
Notifications & User Activity
Đếm thông báo thời gian thực qua SignalR (NotificationHub)
Trạng thái online/offline qua UserStatusHub
Theo dõi hoạt động người dùng và gợi ý kết bạn (follows, recommended friends, ratings)
Machine Learning Integration (ML.NET)
ML.NET models cho phân loại bài viết và bình luận (BlogPostModelInput/Output, BlogCommentModelInput/Output)
Nạp model bằng PredictionEnginePool từ BlogPostMLModel.zip và BlogCommentMLModel.zip
Hỗ trợ workflow duyệt bài/bình luận ở khu vực Administration
Background Jobs & Maintenance
Hangfire cho các job định kỳ: gợi ý bạn bè, dọn dẹp log hoạt động, xóa tin nhắn chat cũ, xóa thông báo cũ
Hangfire Dashboard (chỉ môi trường non‑production), giới hạn truy cập theo role admin
AI Integration
Tầng AI qua IAiService / AiService và IAiRepository / IAiRepository
Cấu hình Groq thông qua GroqSettings trong Program.cs
Media & Cloud Integration
Lưu trữ & xử lý ảnh với Cloudinary (ApplicationCloudinary)
Gửi email bằng SendGrid (EmailSender)
Tích hợp SMS / xác thực điện thoại với Twilio
UI & UX
Nhiều theme và khu vực: Admin dashboard UI, shop‑style front, giao diện private chat
FontAwesome, Bootstrap, admin template trong wwwroot/administration
Client-side storage với Blazored.LocalStorage và Blazored.SessionStorage
Server-Side Blazor components với BlazorStrap
Pagination & Filtering
Phân trang danh sách bằng X.PagedList trong controllers và view components
Phân trang profile, all-users, recommended-users, banned-users, categories/tags
Security & Compliance
Cookie-based auth, antiforgery, role-based authorization
Tích hợp Google reCAPTCHA chống bot
Cấu hình Identity & security stamp để cập nhật roles/claims tức thì
Technology Stack
Runtime & Frameworks
Layer / Purpose	Technology / Package
Runtime	.NET 9.0 (TargetFramework trong CodeSphere.csproj)
Web Framework	ASP.NET Core MVC, Razor Pages, Server-Side Blazor
Identity & Auth	Microsoft.AspNetCore.Identity.EntityFrameworkCore, Microsoft.AspNetCore.Identity.UI
ORM	Microsoft.EntityFrameworkCore.SqlServer, Microsoft.EntityFrameworkCore.Sqlite
Real-time Communication	Microsoft.AspNetCore.SignalR
Background Jobs	Hangfire.Core, Hangfire.AspNetCore, Hangfire.SqlServer
Machine Learning	Microsoft.ML, Microsoft.ML.LightGbm, Microsoft.ML.Recommender, Microsoft.Extensions.ML
Markdown Rendering	Markdig
Pagination	X.PagedList, X.PagedList.Mvc.Core
Cloud Media	CloudinaryDotNet
Email Delivery	SendGrid
SMS / Phone Verification	Twilio
Office/Excel Export	EPPlus
Frontend Rich Text Editor	TinyMCE (NuGet TinyMCE + wwwroot/lib/tinymce)
UI / CSS Framework	Bootstrap, FontAwesome
Blazor UI Helpers	BlazorStrap, Blazored.LocalStorage, Blazored.SessionStorage
HTML Sanitization	HtmlSanitizer
External Auth Providers	Microsoft.AspNetCore.Authentication.Facebook, Google, Twitter
HTTP & JSON	System.Net.Http.Json
Other Utilities	morelinq, Microsoft.VisualStudio.Web.CodeGeneration.Design, StyleCop.Analyzers
System Architecture
Ứng dụng được tổ chức theo kiến trúc MVC nhiều lớp, với Areas cho từng module lớn và phân tầng rõ ràng:

Controllers: xử lý HTTP request, binding và orchestration
Repositories: truy cập dữ liệu và nghiệp vụ theo từng domain (blog, user, chat, notifications, admin, AI, ...)
Services: xử lý nghiệp vụ cross-cutting (AdminAccountService, AiService, ...)
Data Layer: ApplicationDbContext sử dụng EF Core + Identity
ViewModels: lớp DTO cho views và components
Hubs: SignalR hubs cho real-time chat & notifications
ML Models: Input/Output types cho ML.NET + PredictionEngine pools
High-Level Flow (Conceptual)
Client (Browsers, Blazor UI)
        ↓
ASP.NET Core MVC / Razor Pages / Blazor
        ↓
Controllers & ViewComponents
        ↓
Services (AiService, AdminAccountService, etc.)
        ↓
Repositories (Blog, User, Chat, Notifications, Admin, AI, ...)
        ↓
Entity Framework Core (ApplicationDbContext)
        ↓
SQL Server Database
Real-time (chat, notifications, presence) đi qua SignalR hubs, còn các tác vụ ML sử dụng PredictionEnginePool trong repositories/controllers xử lý nội dung pending.

Core Architecture Layers
Controllers
Nằm dưới Controllers/ và Areas/*/Controllers/.
Nhiệm vụ: nhận request, binding models, validation cơ bản, gọi repositories/services và trả về views.
Ví dụ:
Blog & content: BlogController, CategoryController, TagController, UserPostsController, AllCategoriesController
Hồ sơ người dùng: ProfileController
AI: AiController (dùng Markdig + AiService)
Trong Areas:
Administration: PendingPostsController, PendingCommentsController, ReportsController, DashboardController, cùng các controller quản lý emoji, chat theme, chat sticker, holiday theme, ...
Editor: PostController, CommentController, AddCategoryController, EditCategoryController
PrivateChat: PrivateChatController, CollectStickersController
UserNotifications: NotificationController
Services
Dưới Services/:
AdminAccountService: seed/đảm bảo tồn tại các tài khoản có role đặc biệt (admin/editor/contributor).
IAiService / AiService: đóng gói logic gọi AI (Groq, HTTP client).
Được đăng ký DI bằng AddTransient và sử dụng trong controllers hoặc startup logic.
Repositories (Repository Pattern)
Dưới Repositories/ và Areas/*/Repositories/:
Blog & Content: IBlogRepository, IPostRepository, ICategoryRepository, ITagRepository, IBlogComponentRepository, IUserPostsRepository, ...
Profile & Users: IProfileRepository, các repository phân trang: ProfileActivitiesRepository, ProfileFollowersRepository, ProfileFollowingRepository, ProfileFavoritesRepository, ProfilePendingPostsRepository, ProfileBannedPostsRepository, v.v.
Admin & Moderation (trong Areas/Administration/Repositories):
Pending posts/comments: IPendingPostsRepository, IPendingCommentsRepository
Blog addons: IBlogAddonsRepository
Emojis & Stickers: IAddEmojisRepository, IEditEmojiRepository, IDeleteEmojiRepository, IEditEmojiPositionRepository, IAllEmojisRepository, IAddChatThemeRepository, IEditChatThemeRepository, IDeleteChatThemeRepository, IAddChatStickerRepository, IAddChatStickerTypeRepository, IEditChatStickerRepository, IDeleteChatStickerRepository, IAllChatStickersRepository, v.v.
User penalties & dashboard: IUsersPenaltiesRepository, IDbUsageRepository, IDashboardRepository
Reports: IBlogPostReport
Users information & administrators: IUsersInformationRepository, IAllAdministratorsRepository
Private Chat: IPrivateChatRepository, ICollectStickersRepository, IDeleteMessages
Notifications: INotificationRepository, INotificationDbUsage (NotificationDbUsage)
Khác: IHomeRepository, IContactRepository, IAiRepository, IRecommendedFriends, IAllActivities, ...
Data Models (Domain Models)
Nằm trong Models/ và Areas/*/Models/:
Blog: Post, PostImage, Comment, Category, Tag, PostTag, PostLike, FavouritePost, PendingPost, BlockedPost.
User & Location: ApplicationUser, ApplicationRole, ApplicationUserRole, Country, State, City, ZipCode, CountryCode, UserRating, FollowUnfollow, UserAction, RecommendedFriend.
Enums: PostStatus, CommentStatus, Gender, UserActionStatus, UserPostsFilter, ProfileTab, AllUsersTab.
PrivateChat: ChatTheme, ChatMessage, ChatImage, Group, UserGroup, Emoji, EmojiSkin, StickerType, Sticker, FavouriteStickers, QuickChatReply, EmojiType.
User Notifications: UserNotification, NotificationStatus, NotificationType.
Administration: HolidayTheme, HolidayIcon, enum Roles.
ViewModels
Dưới ViewModels/ và Areas/*/ViewModels/:
DTO cho:
Tạo/sửa bài viết: CreatePostInputModel, EditPostInputModel, CreatePostIndexModel
Trang hồ sơ và hoạt động: HomeViewModel, profile view models, các pagination view models
Category/Tag views: CategoryPageViewModel, CategoryPageCategoryViewModel, v.v.
Sửa comment: EditCommentInputModel
Users: LoginInputModel, ManageAccountInputModel, UsersViewModel, ZipCodeViewModel, StateViewModel, ...
Contacts: ContactInputModel
Administration: rất nhiều view models cho emoji, chat sticker, chat theme, holiday theme, penalties, reports, dashboard, v.v.
PrivateChat ViewModels:
PrivateChatViewModel, LoadMoreMessagesViewModel, ChatStickerViewModel, ChatEmojiViewModel, ChatStickerTypeViewModel, QuickChatReplyViewModel, SendFilesResponseViewModel
Collect stickers view models: CollectStickersBaseModel, CollectStickersStickerViewModel, CollectStickersStickerTypeViewModel, ...
Data Layer
Data/ApplicationDbContext
Kế thừa IdentityDbContext<ApplicationUser, ApplicationRole, ...>
Khai báo DbSet<> cho toàn bộ entity domain (blog, user, notifications, private chat, holiday theme, ...).
Cấu hình:
Tên bảng Identity: ApplicationUsers, ApplicationRoles, ApplicationUsersRoles.
Quan hệ và delete behavior cho:
Post ↔ ApplicationUser, Category, Comment, PostImage
City ↔ State, Country, ZipCodes
ApplicationUser ↔ ZipCode, CountryCode, Country, State, City, UserRoles, RecommendedFriends, Followers, Following
Composite keys cho PostTag, PostLike, FollowUnfollow, RecommendedFriend, FavouritePost, v.v.
Migrations trong Migrations/, bao gồm snapshot ApplicationDbContextModelSnapshot.
SignalR Hubs
PrivateChatHub
Quản lý tham gia group, gửi tin nhắn text/sticker, cập nhật thông báo, typing indicator.
NotificationHub
Lấy và gửi số lượng thông báo chưa đọc cho user hiện tại.
UserStatusHub
Theo dõi danh sách user online trong bộ nhớ, broadcast UserIsOnline/UserIsOffline.
ML Models
MlModels/PostModels/BlogPostModelInput/Output
MlModels/CommentModels/BlogCommentModelInput/Output
Đăng ký trong Program.cs qua:
AddPredictionEnginePool<BlogPostModelInput, BlogPostModelOutput>().FromFile("MlModels/PostModels/BlogPostMLModel.zip");
AddPredictionEnginePool<BlogCommentModelInput, BlogCommentModelOutput>().FromFile("MlModels/CommentModels/BlogCommentMLModel.zip");
Được sử dụng trong các repository/controller của Administration cho pending posts/comments.
Feature Modules (Areas)
Administration
Quản trị nội dung và hệ thống:
Dashboard, thống kê, quản lý roles.
Moderation: PendingPostsController, PendingCommentsController, dùng ML hỗ trợ duyệt.
Blog addons: quản lý categories/tags (add/edit/remove).
Emoji & Stickers: CRUD emoji, emoji skins, sticker types, stickers, vị trí emoji, xóa theo type.
Chat & Holiday Themes: quản lý ChatTheme, HolidayTheme, HolidayIcon.
User penalties & users information: banned users, penalties, all admins.
Site reports: báo cáo bài viết/bình luận.
Editor
Khu vực dành cho editor:
Quản lý bài viết, bình luận (PostController, CommentController).
Quản lý categories (AddCategoryController, EditCategoryController).
Sử dụng repositories EditorPostRepository, EditorCommentRepository, AddCategoryRepository, EditCategoryRepository.
PrivateChat
Giao diện và nghiệp vụ chat riêng tư:
PrivateChatController, CollectStickersController, views cho chat và collect stickers.
Models cho message, image, theme, emoji, sticker, favorite sticker, quick reply, group, user group.
Repositories PrivateChatRepository, CollectStickersRepository, DeleteMessages.
Tích hợp chặt chẽ với PrivateChatHub, NotificationHub, UserStatusHub.
UserNotifications
Quản lý thông báo:
NotificationController + views (Areas/UserNotifications/Views/Notification/Index.cshtml).
UserNotification model, enums NotificationStatus, NotificationType.
NotificationRepository, NotificationDbUsage, gắn với background job dọn dẹp.
Identity
Dưới Areas/Identity/:
Razor Pages mặc định của ASP.NET Core Identity (Register, Login, Manage, 2FA, PersonalData, v.v.)
Dùng ApplicationDbContext và ApplicationUser.
Machine Learning Integration
Packages: Microsoft.ML, Microsoft.ML.LightGbm, Microsoft.ML.Recommender, Microsoft.Extensions.ML.
Models:
Blog Post: BlogPostModelInput / BlogPostModelOutput
Comment: BlogCommentModelInput / BlogCommentModelOutput
Model files: BlogPostMLModel.zip, BlogCommentMLModel.zip (copy-to-output trong .csproj).
Luồng sử dụng:
Được đăng ký qua AddPredictionEnginePool trong Program.cs, nạp từ file model.
Dùng trong các repository/controller của Administration để đánh giá bài viết/bình luận pending, hỗ trợ quyết định duyệt từ phía admin.
Real-Time Communication
SignalR Hubs:
PrivateChatHub – chat 1-1, group, gửi message/sticker, trigger notifications.
NotificationHub – đếm notifications chưa đọc và push về client.
UserStatusHub – online presence (theo dõi danh sách user online và broadcast).
Endpoints trong Program.cs:
endpoints.MapHub<PrivateChatHub>("/privateChatHub");
endpoints.MapHub<NotificationHub>("/notificationHub");
endpoints.MapHub<UserStatusHub>("/userStatusHub");
Database Design
DbContext: ApplicationDbContext
Kế thừa IdentityDbContext, rename bảng Identity.
Quan hệ:
Post ↔ ApplicationUser, Category, Comment, PostImage
City ↔ State, Country, ZipCode
ApplicationUser ↔ ZipCode, CountryCode, Country, State, City, UserRoles, RecommendedFriends, Followers, Following
Composite keys: PostTag, PostLike, FollowUnfollow, RecommendedFriend, FavouritePost, ...
Migrations: trong Migrations/, có ApplicationDbContextModelSnapshot và các migration như Initial, FixDatabase2, ...
Database Provider:
Chính: SQL Server (UseSqlServer(DefaultConnection) trong Program.cs).
SQLite được tham chiếu cho tooling, nhưng SQL Server là provider thực tế.
Project Structure
CodeSphere/
  CodeSphere.csproj
  Program.cs
  Data/
    ApplicationDbContext.cs
    ApplicationDbContextFactory.cs
  Models/
    Blog/ (Post, Comment, Category, Tag, ... )
    User/ (ApplicationUser, ApplicationRole, Location, Rating, ... )
    Enums/ (PostStatus, CommentStatus, Gender, ...)
  Areas/
    Administration/
      Controllers/
      Models/
      Repositories/
      ViewModels/
      Views/
    Editor/
      Controllers/
      Repositories/
      ViewModels/
      Views/
    PrivateChat/
      Controllers/
      Models/
      Repositories/
      ViewModels/
      Views/
    UserNotifications/
      Controllers/
      Models/
      Repositories/
      ViewModels/
      Views/
    Identity/
      Pages/Account/...
  Controllers/
    BlogController, CategoryController, TagController,
    ProfileController, UserPostsController, AllCategoriesController, AiController, ...
  Repositories/
    BlogRepositories/
    CategoryRepositories/
    ProfileRepositories/
    UserPostRepositories/
    HomeRepositories/
    ContactRepositories/
    TagRepositories/
    AiRepositories/
    CloudRepositories/
    UserActivitesDbUsage/
    RecommendedFriendsRepositories/
    AllCategories/
    ...
  Services/
    AdminAccountService.cs
    AiService.cs
    IAiService.cs
  Hubs/
    PrivateChatHub.cs
    NotificationHub.cs
    UserStatusHub.cs
  MlModels/
    PostModels/
    CommentModels/
  ViewModels/
    Blog/
    PostViewModels/
    Users/
    Contacts/
    Pagination/
    Profile/
    ...
  Views/
    Shared/, Blog/, Post/, Home/, AllCategories/, ...
  wwwroot/
    css/, js/, images/
    administration/ (Bootstrap, FontAwesome, admin template)
    privateChat/ (chat CSS)
    shop/ (front/theme)
    lib/tinymce/ (TinyMCE)
Installation Guide
Prerequisites
.NET SDK 9.0 trở lên
SQL Server cho app & Hangfire
(Tuỳ chọn) Node/npm nếu muốn build lại front-end assets
Cấu hình hợp lệ cho:
Cloudinary
SendGrid
Twilio
Google/Facebook auth
Groq (AI)
Google reCAPTCHA
Steps
Clone repo
git clone <your-repo-url>
cd CodeSphere/CodeSphere
Cấu hình appsettings
Thiết lập trong appsettings.json :

ConnectionStrings:DefaultConnection
Authentication:Facebook:AppId, AppSecret
Authentication:Google:ClientId, ClientSecret
Cloudinary:CloudName, ApiKey, ApiSecret
Twilio:AccountSID, AuthToken, section Twilio
Section Groq cho GroqSettings
Section GoogleReCAPTCHA cho ReCaptchSettings
Cấu hình SendGrid theo EmailSender.
Apply migrations
dotnet ef database update
Run
dotnet run
Mở URL hiển thị trên console (https://localhost:5001).

Configuration
Các phần cấu hình chính :

Database: ConnectionStrings:DefaultConnection dùng cho ApplicationDbContext và Hangfire.
Identity & Cookies: cấu hình password, login path, access denied path, sliding expiration.
Social Auth: Authentication:Facebook, Authentication:Google.
Cloudinary: Cloudinary:CloudName, ApiKey, ApiSecret.
Twilio: Twilio:AccountSID, AuthToken, Twilio section cho TwilioVerifySettings.
Groq (AI): section Groq cho GroqSettings.
Google reCAPTCHA: section GoogleReCAPTCHA cho ReCaptchSettings.
Hangfire: sử dụng DefaultConnection, dashboard chỉ bật khi không phải Production, path /Administration/UsersPenalties/HangFire.
Security
Authentication
ASP.NET Core Identity + external login (Facebook, Google).
Cookie-based authentication với sliding expiration.
Authorization
Role-based, enum roles trong Areas/Administration/Models/Enums/Roles.
Hangfire dashboard chỉ cho phép user có role admin, và chỉ khi không Production.
Antiforgery
Đặt HeaderName = "X-CSRF-TOKEN".
Security Stamp
Validation interval = 0 phút, cập nhật role/claims ngay lập tức.
Session & State
Session timeout 30 phút, HttpOnly = true, IsEssential = true.
reCAPTCHA
Cấu hình GoogleReCAPTCHA để chống bot/spam.
HTML Sanitization
Dùng HtmlSanitizer để làm sạch HTML cho nội dung rich text.
Static Assets
wwwroot bao gồm:

Site CSS: css/site.css, header.css, holidayThemeSheet.css, private chat CSS.
Admin Template: wwwroot/administration/ (Bootstrap, FontAwesome, JS, DataTables demo).
Shop/Front UI: wwwroot/shop/ (images, CSS, carousel, animation).
TinyMCE: wwwroot/lib/tinymce/ (core, plugins, skins).
Private Chat Assets: wwwroot/privateChat/ (chatStickersSheet.css, privateChat.css).
