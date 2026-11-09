---
name: testcase-script-generator
description: 根据用户提供的测试用例（Steps + Expected Result），生成符合项目规范的 Playwright 测试脚本，包括 spec 文件、所需的 Page Object 方法、以及 fixture 注册。
---

# 测试用例 → Playwright 脚本生成器

## 适用场景（何时调用）

当用户提供测试用例并希望生成测试脚本时调用：
- 用户粘贴了测试用例的 Steps + Expected Result
- 用户提供了测试用例文档路径或截图
- 用户说"帮我写/生成/作成这个测试用例的脚本"

## 输入要求（最少信息）

请一次性提供：

1. **测试用例内容**（必须）：Steps + Expected Result，格式不限（表格、文字、截图均可）
2. **测试用例 ID**（必须）：如 T301、T715 等，用于文件命名和常量前缀
3. **目标应用**（必须）：`@Server`（KMServer）或 `@Web`（KMWeb）
4. **所属模块**（建议）：如 jobList、administration、printProductsAndServices 等
5. **已有 Page Object 路径**（可选）：如果已知涉及哪些 Page Object 文件

若信息不足，主动询问后再生成，不要猜测。

## 生成流程（必须按顺序执行）

### Phase 1：分析测试用例

1. **提取关键信息**：
   - 测试目的（用户故事）
   - 前置条件（beforeEach 需要什么）
   - 每个 Step 的动作和预期结果
   - 清理需求（afterEach 需要什么）

2. **识别涉及的 UI 页面/组件**：
   - 用例中提到了哪些页面（Job List、Import Dialog、Configuration 等）
   - 涉及哪些操作（点击、输入、选择、上传、断言等）

### Phase 2：搜索已有 Page Object

**在生成新方法前，必须先搜索项目中已有的 Page Object 和方法。**

搜索策略：
```
1. 按页面名称搜索：grep -r "class.*extends BaseFunction" test/gui/kmserver/api/pages/
2. 按功能关键词搜索：grep -r "clickSave\|uploadFile\|selectProduct" test/gui/kmserver/api/pages/
3. 检查 testSetup.ts 中已注册的 fixture 名称（第 97-244 行的 MyFixtures 类型）
4. 检查 BaseFunction.ts 中已有的通用方法（loading、clickOkButtonDialog0 等）
5. 检查 BasePage.ts 中是否已有可复用的底层能力；若没有，优先补到 BasePage.ts，不要直接在页面对象里写原生 `this.page`
```

已注册的主要 fixture 映射表（按需查阅 testSetup.ts）：

| fixture 名称 | 类 | 路径 |
|---|---|---|
| `loginManager` | LoginServer | `@serverPages/Login` |
| `overviewServer` | OverviewServer | `@serverPages/OverviewServer` |
| `jobList` | JobList | `@serverPages/jobList/JobList` |
| `jobImport` | JobImport | `@serverPages/jobList/JobImport` |
| `jobEditor` | JobEditor | `@serverPages/jobList/JobEditor` |
| `jobViewDialog` | JobViewDialog | `@serverPages/jobList/JobViewDialog` |
| `servicesView` | Services | `@serverPages/PrintProductsAndServices/Services/Services` |
| `productsView` | PrintProducts | `@serverPages/PrintProductsAndServices/PrintProducts/PrintProducts` |
| `printProductDLG` | PrintProductDLG | `@serverPages/PrintProductsAndServices/PrintProducts/PrintProductsDialog` |
| `deliveryTypeView` | DeliveryTypes | `@serverPages/PrintProductsAndServices/DeliveryTypes/DeliveryTypes` |
| `deliveryTypeDlgView` | DeliveryTypesDialog | `@serverPages/PrintProductsAndServices/DeliveryTypes/DeliveryTypesDialog` |
| `generalSettings` | GeneralSettings | `@serverPages/administrator/GeneralSettings` |
| `addresses` | Addresses | `@serverPages/Workflow/Addresses` |
| `addressesDialog` | AddressesDialog | `@serverPages/Workflow/AddressesDialog` |
| `hotFolder` | HotFolder | `@serverPages/Workflow/HotFolder` |
| `signatures` | Signatures | `@serverPages/Media/Signatures` |
| `signaturesLayoutTab` | SignaturesLayoutTab | `@serverPages/Media/SignaturesLayoutTab` |
| `signaturesDialog` | SignaturesDialog | `@serverPages/Media/SignaturesDialog` |
| `rightsManagement` | RightsManagement | `@serverPages/administrator/roles/RightsManagement` |
| `rightsManagementDlg` | RightsManagementDialog | `@serverPages/administrator/roles/RightsManagementDialog` |
| `paperFormats` | PaperFormats | `@serverPages/Media/PaperFormats` |
| `printers` | Printers | `@serverPages/PrinterAndClusters/Printers` |
| `dashboard` | Dashboard | `@serverPages/Dashboard/Dashboard` |
| `addSmartFolderDialog` | AddSmartFolderDialog | `@serverPages/jobList/AddSmartFolderDialog` |
| `VirtualPrinter` | VirtualPrinter | `@serverPages/Workflow/VirtualPrinter` |
| `VirtualPrinterDialog` | VirtualPrinterDialog | `@serverPages/Workflow/VirtualPrinterDialog` |
| `preflight` | Preflight | `@serverPages/Workflow/Preflight` |
| `orderInformation` | OrderInformationTab | `@serverPages/jobList/OrderInformationTab` |
| `login` | Login (Web) | `@pages/Login` |
| `globalView` | OverviewWeb | `@pages/OverviewWeb` |
| `account` | Account | `@pages/manager/administration/Account` |
| `dashboard` | Dashboard (Web) | `@pages/manager/Dashboard` |
| `command` | Api (WebCommands) | `@api/webCommands` |
| `serverCommand` | ServerCommands | `@serverApi/commands` |

常用枚举 fixture：

| fixture 名称 | 枚举 | 来源 |
|---|---|---|
| `LayoutType` | LayoutType | JobSetting |
| `ImpositionType` | ImpositionType | JobSetting |
| `columns` | COLUMNS | JobList |
| `folder` | FOLDER | JobList |
| `DayType` | Day_Type | DeliveryTypesDialog |
| `chapterCheckBox` | ChapterCheckBox | ChaptersTab |
| `filterPosition` | FilterPosition | AddSmartFolderDialog |
| `Role` | Role | UserAccountConfigServerDialog |

### Phase 3：生成脚本

#### 3.1 确定文件路径

根据模块和应用确定 spec 文件路径：
- Server 测试：`test/gui/kmserver/tests/{module}/Test{Desc}T{ID}.spec.ts`
- Web 测试：`test/gui/kmweb/tests/{module}/Test{Desc}T{ID}.spec.ts`

#### 3.2 生成 spec 文件

严格遵循以下模板结构：

```typescript
/*
    Created by {author}
    Windows11 64bit EN
    Playwright@1.45.1
    TestBrowser: Chrome(Version  135.0.7049.114)
    TestFluxVersion:{version}
*  *********************************
As {role}
I want to {action}
So that {purpose}
 */
import { test, expect } from '@setup/testSetup'

let RANDOM = Math.floor(Math.random() * 1000).toString().padStart(3, '0');
const UNIQUE_NAME = "T{ID}_prefix_" + RANDOM;

test.describe('@suite{N} @Server', () => {
    test.beforeEach(async ({ loginManager, ... }) => {
        // 前置条件：登录、创建必要数据、登出
    });

    test.afterEach(async ({ loginManager, ... }) => {
        // 清理：登录、删除测试数据、登出
    });

    test('Test{Desc}T{ID}', async ({ loginManager, ... }) => {
        test.setTimeout({timeout});

        //Step1: {description}
        // ... 动作和断言 ...

        //Step2: {description}
        // ... 动作和断言 ...
    })
})
```

#### 3.3 生成 Page Object 方法（如需）

当用例涉及的交互在已有 Page Object 中找不到对应方法时，生成新方法。

#### 3.3.1 Page Object / Helper 分层硬规则

生成或补充 Page Object / Helper 方法时，必须遵守：

- 禁止在 `BasePage` 之外新增任何直接 `this.page` / `.page` 调用
- 禁止在 Page Object / Helper 中直接写 `this.page.locator(...)`、`this.page.getBy...(...)`、`this.page.context()`、`locator().count()` 等原生 Playwright 底层操作
- 若所需能力在 `BasePage` 中不存在，必须先补充 `BasePage` 通用方法，再由具体页面对象调用
- Page Object 方法只负责业务语义组合，例如 `waitForTokenAdded()`、`getTokenCount()`，其底层实现应基于 `BasePage` 提供的方法完成

Page Object 文件结构：
```typescript
import BaseFunction from '../BaseFunction';

// 选择器常量 — 大写下划线，优先 data-testid
const MY_BUTTON_CSS = '[data-testid="my-button"]';
const MY_DIALOG_CSS = '[data-testid="my-dialog"]';
const DYNAMIC_ITEM_CSS = '[data-testid="item-%s"]';  // %s 用于动态替换

export default class MyPage extends BaseFunction {

    async clickMyButton() {
        await this.clickByCss(MY_BUTTON_CSS);
    }

    async isMyDialogVisible(): Promise<boolean> {
        return await this.isVisibleByCss(MY_DIALOG_CSS);
    }

    async clickDynamicItem(name: string) {
        const css = DYNAMIC_ITEM_CSS.replace('%s', name);
        await this.clickByCss(css);
    }
}
```

#### 3.4 注册 fixture（如需新增 Page Object）

在 `test/test/gui/testSetup.ts` 中添加：

```typescript
// 1. import 新的 Page Object
import MyNewPage from '@serverPages/path/MyNewPage'

// 2. 在 MyFixtures 类型中添加
type MyFixtures = {
  // ... 现有 fixture ...
  myNewPage: MyNewPage;
}

// 3. 在 testSetup.extend 中注册
myNewPage: createPageFixture(MyNewPage),
```

### Phase 4：自检清单（生成后必须检查）

生成的脚本必须通过以下检查，不通过则修改后再输出：

| 检查项 | 要求 |
|---|---|
| 文件命名 | `Test{Desc}T{ID}.spec.ts` |
| 导入来源 | `import { test, expect } from '@setup/testSetup'` |
| RANDOM 后缀 | 所有测试数据名拼接 RANDOM |
| describe 标签 | `@suite{N} @Server` 或 `@Web` |
| Step 注释 | 每个步骤前有 `//Step{N}: {描述}` |
| 断言 message | 每个 expect 带 `'Step{N}: {描述}'` |
| test.setTimeout | 设置合理超时（默认 100000，复杂流程 200000+） |
| 无 page 直接操作 | spec 中不出现 `page.locator()`、`page.waitForSelector()` |
| POM 无 `.page` 直接调用 | `BasePage` 之外不出现 `this.page` / `.page.locator()` / `locator().count()` |
| 无方法定义 | spec 中不定义 function/箭头函数/class |
| 无固定等待 | spec 中不出现 `page.waitForTimeout()` |
| fixture 注入 | 所有 Page Object 通过参数获取，不手动 new |
| 选择器规范 | 优先 data-testid，禁止 nth-child/超长 xpath |
| 断言模式 | 动态 UI 用 web-first assertion 或 BasePage.expect* |
| 清理覆盖 | afterEach 清理所有创建的数据 |

## 输出结构（必须包含）

### 1. 分析摘要

简述用例涉及的页面、操作类型、已有方法覆盖情况。

### 2. 覆盖情况

列出每个 Step 中：
- 已有 Page Object 方法可直接使用的方法
- 需要新增的 Page Object 方法（附完整代码）
- 需要新增的 fixture 注册（附修改位置）

### 3. 生成的文件

按优先级输出：
1. **spec 文件**（完整代码，可直接复制使用）
2. **新增 Page Object 方法**（如有，给出完整方法代码和插入位置）
3. **fixture 注册修改**（如有，给出 testSetup.ts 的具体修改）

### 4. 执行说明

- 如何运行：`npx playwright test Test{Desc}T{ID}.spec.ts --project=chromium`
- 依赖的测试数据文件（如有）
- 前置条件说明

## 代码风格约束（硬性要求）

### spec 文件中：
- **禁止**定义任何 function/箭头函数/class
- **禁止**直接操作 `page` 对象
- **禁止**调用 BasePage 底层方法（`clickByCss`、`waitForSelector` 等）
- **禁止**使用 `page.waitForTimeout()`
- **禁止**手动 `new` Page Object
- **必须**通过 fixture 参数获取所有 Page Object
- **必须**每个 expect 带描述性 message
- **必须** Step 注释与断言 message 编号一致

### Page Object 文件中：
- **必须**继承 `BaseFunction`（KMServer）或 `BaseFunction`（KMWeb）
- **必须**选择器定义为模块级 `const`，大写下划线命名
- **必须**方法名体现业务语义（`clickSaveButton` 而非 `clickButton`）
- **必须**方法具备通用性（参数化而非重复方法）
- **必须**优先复用 `BasePage` 已封装的底层能力；若缺少能力，先补 `BasePage`
- **禁止** `is*`/`get*` 方法 catch 后无上下文 return false/null
- **禁止**在 `BasePage` 之外直接出现 `this.page` / `.page`
- **禁止**直接使用原生 Playwright 底层调用，如 `locator()`、`count()`、`getAttribute()`、`context()`，除非该实现位于 `BasePage`
- **禁止**参数或返回值暴露 `Page`/`Locator` 类型给 spec 层

### 数据命名规范：
- 常量前缀：`T{ID}_`（如 `T301_PRODUCT_NAME`）
- RANDOM：`let RANDOM = Math.floor(Math.random() * 1000).toString().padStart(3, '0')`
- 拼接方式：`const NAME = "T{ID}_prefix_" + RANDOM`

## 示例：完整生成流程

**用户输入：**
```
用例 ID: T999
应用: @Server
模块: jobList
用例内容:
- Step1: 打开 Job List，上传 single_page.pdf
- Step2: 在 Import Dialog 中输入 Job Title，点击 Create
- Step3: 验证 Job 出现在 Job List 中
- Expected: Job 成功创建并显示在列表中
```

**生成输出：**

分析：涉及 JobList 和 JobImport 两个已有 fixture，所有方法已存在（uploadFile、waitImportReady、writeJobTitle、clickCreateButton、isJobInJobList），无需新增 Page Object。

```typescript
/*
    Created by AI Generator
    Windows11 64bit EN
    Playwright@1.45.1
    TestBrowser: Chrome(Version  135.0.7049.114)
    TestFluxVersion:10.4.0
*  *********************************
As administrator
I want to upload a PDF file and create a job
So that the job appears in the job list
 */
import { test, expect } from '@setup/testSetup'

let RANDOM = Math.floor(Math.random() * 1000).toString().padStart(3, '0');
const JOB_TITLE = "T999_job_" + RANDOM;
const FILE_NAME = 'single_page.pdf';

test.describe('@suite1 @Server', () => {
    test.afterEach(async ({ loginManager, jobList, overviewServer }) => {
        await loginManager.adminLogin();
        await jobList.loading();
        await overviewServer.clickJobList();
        await jobList.clearJobList();
        await loginManager.logout();
    })

    test('TestUploadAndCreateJobT999', async ({ loginManager, jobList, overviewServer, jobImport, globalView }) => {
        test.setTimeout(100000);

        //Step1: Open Job List and upload single_page.pdf
        await loginManager.adminLogin();
        await overviewServer.clickJobList();
        await jobList.uploadFile(FILE_NAME);
        await jobImport.waitImportReady();

        //Step2: Enter Job Title and click Create
        await jobImport.selectInfoTab();
        await jobImport.writeJobTitle(JOB_TITLE);
        await jobImport.clickCreateButton();
        await jobImport.waitAfterCreateImportDone();

        //Step3: Verify the job appears in the Job List
        await overviewServer.clickJobList();
        await globalView.loading();
        expect(await jobList.isJobInJobList(JOB_TITLE), 'Step3: The job should appear in the job list').toBeTruthy();
    })
})
```

无需新增 Page Object 方法，无需修改 testSetup.ts。

## 常见陷阱（生成时必须避免）

| 陷阱 | 正确做法 |
|---|---|
| spec 中写了 `page.locator('#btn').click()` | 改用 `pageObject.clickMyButton()` |
| spec 中定义了 `function waitAndClick()` | 方法下沉到 Page Object |
| 断言没有 message | 每个 expect 加 `'Step{N}: description'` |
| Step 编号不连续/不一致 | 注释和 message 编号必须对应 |
| 测试数据硬编码 | 拼接 RANDOM 后缀 |
| afterEach 遗漏清理 | 清理所有 beforeEach 和 test 中创建的数据 |
| 直接 new Page Object | 通过 fixture 参数获取 |
| 等待用 `waitForTimeout` | 用 `waitForShowByCss`、`loading()` 等显式等待 |
| `isDialogVisible()` catch 后 return false | 让异常冒泡或保留上下文 |
| 动态 UI 用 `expect(await isXxx()).toBeTruthy()` | 改用 web-first assertion |
| 在 Page Object 中写 `return await this.page.locator(selector).count()` | 先在 `BasePage` 中封装 `getCountByCss()`，再由 Page Object 调用 |
| 在 Page Object 中写 `await this.page.getByTestId('xxx').click()` | 先复用或补充 `BasePage` 的 `clickByDataTestId()` |
