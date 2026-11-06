# Checklist input cho 33 tools My Work Center

Danh sách này được đối chiếu từ `swagger-actions.json` hiện tại. Đây là checklist cấu hình cần kiểm tra, không phải xác nhận rằng mọi tool trong tenant Microsoft đang bị lỗi. Chỉ tool Create task trong ảnh đã được nhìn thấy có Custom value trống; cấu hình các tool khác chưa được truy cập trực tiếp.

## Cách rà từng tool

1. Trong Build, mở thẻ primary agent → Tools → chọn tool → Inputs.
2. Input lấy từ yêu cầu/ngữ cảnh: chọn Dynamically fill with AI nếu lựa chọn này có trong dropdown. Custom value dùng khi chủ động gắn giá trị cố định, biến hoặc công thức hợp lệ; không để trống input bắt buộc.
3. Dùng + Add input để thêm các trường tùy chọn cần hỗ trợ. Có đầy đủ trường bắt buộc chưa có nghĩa tool đã truyền được hạn, ưu tiên, project hay ID workspace.
4. Không cố định một taskId/projectId/status/date cho mọi lượt. Các ID được lấy từ kết quả tạo hoặc đọc dữ liệu trước đó; không đoán ID hay bắt người dùng đi tìm ID kỹ thuật.
5. Chỉ gửi các trường được yêu cầu thay đổi. Trường tùy chọn không dùng thì bỏ khỏi lần gọi, không điền chuỗi rỗng, chuỗi "null" hoặc giá trị mặc định để thay thế. clearFields chỉ dùng khi người dùng muốn xóa giá trị.
6. Save tool, kiểm tra trong Preview bằng các bản ghi thử do bạn chủ động chọn, rồi Deploy → Publish để chatbot website nhận cấu hình agent mới.

## Quy tắc chung

- **requestKey:** cả 27 tools thao tác đều cần. Agent tự tạo một mã duy nhất cho mỗi lệnh và dùng lại đúng mã cùng input khi retry lệnh đó. Không nhập một giá trị cố định cho mọi lệnh; không dùng công thức sinh mã mới mỗi lần retry. Không hỏi người dùng cung cấp requestKey.
- **expectedRevision:** tùy chọn ở phần lớn tools thao tác; là số revision từ lần đọc hiện tại, không cố định. RestoreWorkspaceBackup bắt buộc có trường này.
- **Các ID và commandId:** AI lấy từ output trước đó hoặc tra cứu. taskId khác commandId; itemId của card khác itemId checklist; milestone phải thuộc đúng project.
- **Ngày/giờ:** ngày YYYY-MM-DD, giờ HH:mm theo Asia/Bangkok. Priority 0=không đặt, 1=cao, 2=trung bình, 3=thấp. Task/checklist status: todo/doing/done; GET status=open có nghĩa các task chưa hoàn thành.
- **Completion:** dùng Send results to primary agent to formulate a response. Trong Advanced/Outputs available to the agent and other tools, bảo đảm agent nhận được commandId, status, result (gồm entityIds, revision và snapshotId khi có). queued/running không có nghĩa đã xong; dùng GetCommandStatus và chỉ báo hoàn tất khi succeeded.

## Danh sách đầy đủ

Mỗi dòng có tên hiển thị để tìm trong Tools và Operation ID để đối chiếu connector. Bảng giữ nguyên mọi input trong spec. Với tools thao tác, requestKey/expectedRevision được liệt kê dù cũng đã giải thích ở trên.

| Tool hiển thị / Operation ID | Input bắt buộc | Input tùy chọn — thêm nếu muốn dùng chức năng tương ứng |
| --- | --- | --- |
| **Get dashboard summary** (GetDashboardSummary) | Không có | `date` |
| **List projects and groups** (GetProjects) | Không có | `id`, `q`, `kind`, `status`, `includeArchived`, `limit`, `offset` |
| **List saved tasks** (GetTasks) | Không có | `id`, `q`, `projectId`, `milestoneId`, `status`, `priority`, `dueDate`, `dueFrom`, `dueTo`, `overdue`, `includeWorkspace`, `limit`, `offset` |
| **List machine inventory** (GetMachines) | Không có | `id`, `q`, `status`, `location`, `limit`, `offset` |
| **Create task** (CreateTask) | `requestKey`, `title` | `expectedRevision`, `note`, `dueDate`, `dueTime`, `priority`, `weight`, `projectId`, `milestoneId`, `tags`, `repeat`, `reminderAt`, `includedInProjectProgress`, `clearFields` |
| **Update task** (UpdateTask) | `requestKey`, `taskId` | `expectedRevision`, `title`, `note`, `dueDate`, `dueTime`, `priority`, `weight`, `projectId`, `milestoneId`, `tags`, `repeat`, `reminderAt`, `includedInProjectProgress`, `clearFields` |
| **Set task status** (SetTaskStatus) | `requestKey`, `taskId`, `status` | `expectedRevision`, `completionComment` |
| **Delete task** (DeleteTask) | `requestKey`, `taskId` | `expectedRevision` |
| **Update multiple tasks** (BulkUpdateTasks) | `requestKey`, `taskIds` | `expectedRevision`, `title`, `note`, `dueDate`, `dueTime`, `priority`, `weight`, `projectId`, `milestoneId`, `tags`, `repeat`, `reminderAt`, `includedInProjectProgress`, `clearFields` |
| **Create goal project** (CreateProject) | `requestKey`, `name`, `goal` | `expectedRevision`, `acceptanceCriteria`, `startDate`, `targetDate`, `color`, `clearFields` |
| **Update project** (UpdateProject) | `requestKey`, `projectId` | `expectedRevision`, `name`, `goal`, `acceptanceCriteria`, `startDate`, `targetDate`, `color`, `clearFields` |
| **Set project status** (SetProjectStatus) | `requestKey`, `projectId`, `status` | `expectedRevision`, `goalAccepted` |
| **Archive or unarchive project** (ArchiveProject) | `requestKey`, `projectId`, `archived` | `expectedRevision` |
| **Create or update milestone** (SaveMilestone) | `requestKey`, `projectId` | `expectedRevision`, `milestoneId`, `name`, `outcome`, `dueDate`, `clearDueDate` |
| **Delete milestone** (DeleteMilestone) | `requestKey`, `projectId`, `milestoneId` | `expectedRevision` |
| **Create or update checklist item** (SaveChecklistItem) | `requestKey`, `taskId` | `expectedRevision`, `itemId`, `title`, `note`, `status`, `placeOnBoard` |
| **Delete checklist item** (DeleteChecklistItem) | `requestKey`, `taskId`, `itemId` | `expectedRevision` |
| **Update workspace notes** (UpdateWorkspaceNotes) | `requestKey`, `taskId`, `text`, `mode` | `expectedRevision` |
| **Create or update workspace card** (SaveWorkspaceItem) | `requestKey`, `taskId` | `expectedRevision`, `itemId`, `type`, `subtaskId`, `text`, `note`, `status`, `tone`, `x`, `y` |
| **Remove workspace card** (DeleteWorkspaceItem) | `requestKey`, `taskId`, `itemId` | `expectedRevision` |
| **Connect or disconnect workflow steps** (SetWorkspaceConnection) | `requestKey`, `taskId`, `fromId`, `toId`, `connected` | `expectedRevision` |
| **Create sequential workflow** (CreateWorkspaceWorkflow) | `requestKey`, `taskId`, `steps` | `expectedRevision` |
| **Create or update habit** (SaveHabit) | `requestKey` | `expectedRevision`, `habitId`, `name`, `frequency`, `color` |
| **Set habit completion for a date** (SetHabitCompletion) | `requestKey`, `habitId`, `date`, `completed` | `expectedRevision` |
| **Delete habit** (DeleteHabit) | `requestKey`, `habitId` | `expectedRevision` |
| **Create or update countdown** (SaveCountdown) | `requestKey` | `expectedRevision`, `countdownId`, `name`, `date`, `color` |
| **Delete countdown** (DeleteCountdown) | `requestKey`, `countdownId` | `expectedRevision` |
| **Open and filter My Work Center** (NavigateWorkCenter) | `requestKey`, `view` | `expectedRevision`, `taskId`, `projectId`, `search`, `viewMode`, `status`, `highOnly`, `date`, `calendarMode` |
| **Control focus timer** (ControlFocusTimer) | `requestKey`, `operation` | `expectedRevision`, `taskId`, `mode`, `focusMinutes`, `breakMinutes` |
| **Create saved workspace backup** (CreateWorkspaceBackup) | `requestKey` | `expectedRevision`, `label` |
| **Restore saved workspace backup** (RestoreWorkspaceBackup) | `requestKey`, `snapshotId`, `expectedRevision` | Không có |
| **Get local browser and utility context** (GetAssistantContext) | Không có | `null` |
| **Check local command result** (GetCommandStatus) | `commandId` | Không có |

## Các trường tùy chọn có điều kiện bắt buộc

Các input dưới đây không luôn có dấu * vì cùng một tool hỗ trợ nhiều trường hợp. Agent vẫn phải truyền chúng khi thực hiện đúng trường hợp:

| Tool | Trường phải có trong trường hợp tương ứng |
| --- | --- |
| CreateTask / UpdateTask / BulkUpdateTasks | dueDate khi đặt/đổi hạn; priority khi đặt/đổi ưu tiên; projectId khi gắn dự án; milestoneId khi gắn mốc. Update/bulk phải có ít nhất một thay đổi có ý nghĩa. |
| NavigateWorkCenter | view=task-workspace cần taskId; view=project cần projectId. Muốn lọc/tìm/đổi bố cục phải thêm search, status, viewMode hoặc các trường tương ứng. |
| SaveMilestone | Tạo mới cần name; sửa cần milestoneId cùng trường cần đổi. |
| SaveChecklistItem | Tạo mới cần title; sửa cần itemId cùng title/note/status cần đổi. placeOnBoard=true khi muốn đưa checklist lên bảng. |
| SaveWorkspaceItem | Sửa card cần itemId; tạo subtask card cần type=subtask và subtaskId; tạo step cần type=step và text có nội dung. |
| SaveHabit | Tạo mới cần name; sửa cần habitId cùng trường cần đổi. |
| SaveCountdown | Tạo mới cần name và date; sửa cần countdownId cùng trường cần đổi. |
| SetProjectStatus | status=completed cần goalAccepted=true từ xác nhận thật của người dùng, đồng thời các task đủ điều kiện phải hoàn thành. |
| ControlFocusTimer | taskId khi gắn phiên với một task; mode, focusMinutes, breakMinutes khi yêu cầu đổi chế độ/thời lượng. |
| GetTasks / GetProjects / GetMachines | Thêm q/id/status và các bộ lọc cần dùng. GetTasks cần includeWorkspace khi muốn đọc workspace; projectId khi lọc theo dự án; limit/offset để lấy đủ các trang. |

## Thứ tự cấu hình nên ưu tiên

1. CreateTask: requestKey, title; thêm dueDate, priority, note, projectId và các trường bạn thường dùng.
2. GetTasks: thêm id, q, status, projectId, includeWorkspace, limit, offset và bộ lọc hạn cần dùng.
3. GetCommandStatus: commandId; kiểm tra output entityIds qua result để nối các lệnh tiếp theo.
4. NavigateWorkCenter: requestKey, view; thêm taskId, projectId, search, status, viewMode.
5. UpdateTask / SetTaskStatus: taskId và các trường tương ứng trong bảng.
6. Tiếp tục các tool dự án, checklist/workspace, tiện ích và backup theo nhu cầu.

Trường hợp đã quan sát: CreateTask chỉ gửi requestKey và title nên task được tạo mà không có hạn/ưu tiên. NavigateWorkCenter chỉ gửi requestKey và view=task-workspace nên thiếu taskId. Bổ sung input ở hai tool này là việc cần làm trước.

Tài liệu Microsoft: [quản lý tools Customer Conversation](https://learn.microsoft.com/en-us/microsoft-copilot-studio/agents-experience/tools-manage-customer-conversation), [cách điền input bằng AI hoặc Custom value](https://learn.microsoft.com/en-us/microsoft-copilot-studio/add-tools-custom-agent#inputs).
