# .claude/templates 활용법

## 개요

`.claude/templates` 폴더는 자주 사용하는 코드 패턴을 **재사용 가능한 템플릿**으로 저장하는 곳입니다. 신규 페이지나 기능을 추가할 때 Claude Code가 이 템플릿을 참고하여 일관된 코드를 생성합니다.

## 왜 필요한가?

프로젝트에서 반복적으로 작성하는 코드 패턴이 있습니다:
- CRUD 페이지 (목록, 등록, 수정, 상세)
- API 엔드포인트
- 인증이 필요한 페이지
- 폼 유효성 검증
- 권한 체크

이런 패턴을 템플릿으로 만들면:
- ✅ 일관된 코드 구조 유지
- ✅ 개발 시간 단축
- ✅ 실수 방지
- ✅ 베스트 프랙티스 공유

## 적정 크기 가이드

### 권장 크기
- **개별 템플릿 파일**: 1,000-2,000 토큰 (약 3,000-6,000자)
- **전체 templates 폴더**: 필요한 만큼 (자주 쓰는 패턴만)

### 토큰 수 확인
- [OpenAI Tokenizer](https://platform.openai.com/tokenizer) 사용
- 한글 기준: 글자 수 × 3 ≈ 토큰 수

### 크기 관리 팁
- 템플릿은 필요할 때만 Claude Code가 참조
- 너무 많은 템플릿보다 핵심 패턴만 유지
- 5-10개 정도의 템플릿이 적당

## 실전 예제

### 예제 1: 맑은프레임워크 CRUD 템플릿

**파일: `.claude/templates/crud-list.md`**

```markdown
# 목록 페이지 템플릿 (맑은프레임워크)

## 사용 방법
이 템플릿을 사용하여 새로운 목록 페이지를 생성하세요.
플레이스홀더를 실제 값으로 변경하세요:
- {{entity}}: 엔티티명 (예: User, Board)
- {{table}}: 테이블명 (예: tb_user, tb_board)
- {{fields}}: 검색 필드 (예: name,email)

## JSP 파일: /{{folder}}/{{entity}}_list.jsp

\`\`\`jsp
<%@ include file="/init.jsp" %><%

// 검색 키워드
String keyword = m.rs("keyword");

// 페이징 사용
ListManager lm = new ListManager();
lm.setRequest(request);
lm.setTable("{{table}}");
lm.setListNum(20);

// 검색 조건
if(!"".equals(keyword)) {
    lm.addSearch("{{fields}}", keyword, "LIKE");
}

lm.setOrderBy("id DESC");
DataSet list = lm.getDataSet();
String paging = lm.getPaging();

// 템플릿 설정
p.setLayout("default");
p.setBody("{{folder}}.{{entity}}_list");
p.setVar("keyword", keyword);
p.setVar("paging", paging);
p.setLoop("list", list);
p.display();
%>
\`\`\`

## HTML 템플릿: /html/{{folder}}/{{entity}}_list.html

\`\`\`html
<div class="container">
    <h1>{{entity}} 목록</h1>

    <!-- 검색 폼 -->
    <form method="get" class="mb-3">
        <div class="input-group">
            <input type="text" name="keyword" value="{{keyword}}"
                   class="form-control" placeholder="검색어 입력">
            <button type="submit" class="btn btn-primary">검색</button>
        </div>
    </form>

    <!-- 목록 테이블 -->
    <table class="table">
        <thead>
            <tr>
                <th>ID</th>
                <th>제목</th>
                <th>등록일</th>
            </tr>
        </thead>
        <tbody>
            <!--@loop(list)-->
            <tr>
                <td>{{list.id}}</td>
                <td><a href="{{entity}}_view.jsp?id={{list.id}}">{{list.title}}</a></td>
                <td>{{list.reg_date}}</td>
            </tr>
            <!--/loop(list)-->
        </tbody>
    </table>

    <!-- 페이징 -->
    <div class="text-center">
        {{paging}}
    </div>

    <!-- 등록 버튼 -->
    <div class="text-end">
        <a href="{{entity}}_insert.jsp" class="btn btn-primary">등록</a>
    </div>
</div>
\`\`\`

## DAO 파일: /src/dao/{{entity}}Dao.java

\`\`\`java
package dao;

import malgnsoft.db.DataObject;

public class {{entity}}Dao extends DataObject {
    public {{entity}}Dao() {
        this.table = "{{table}}";
    }
}
\`\`\`
```

**파일: `.claude/templates/crud-insert.md`**

```markdown
# 등록 페이지 템플릿 (맑은프레임워크)

## 사용 방법
Postback 패턴을 사용한 등록 페이지입니다.

## JSP 파일: /{{folder}}/{{entity}}_insert.jsp

\`\`\`jsp
<%@ include file="/init.jsp" %><%

// 유효성 검증 설정
f.addElement("title", null, "required:Y, maxLen:100");
f.addElement("content", null, "required:Y");

// POST 처리 (등록)
if(m.isPost() && f.validate()) {
    {{entity}}Dao dao = new {{entity}}Dao();
    dao.item("title", f.get("title"));
    dao.item("content", f.get("content"));
    dao.item("user_id", userId);
    dao.item("reg_date", m.time());

    if(dao.insert()) {
        m.jsAlert("등록되었습니다.");
        m.jsReplace("{{entity}}_list.jsp");
    } else {
        m.jsError("등록 실패: " + dao.getErrMsg());
    }
    return;
}

// GET 처리 (폼 표시)
p.setLayout("default");
p.setBody("{{folder}}.{{entity}}_form");
p.setVar("is_insert", true);
p.setVar("form_script", f.getScript());
p.display();
%>
\`\`\`

## HTML 템플릿: /html/{{folder}}/{{entity}}_form.html

\`\`\`html
<div class="container">
    <h1>{{entity}} 등록</h1>

    <form method="post">
        <div class="mb-3">
            <label class="form-label">제목</label>
            <input type="text" name="title" class="form-control">
        </div>

        <div class="mb-3">
            <label class="form-label">내용</label>
            <textarea name="content" class="form-control" rows="10"></textarea>
        </div>

        <div class="text-end">
            <!--@if(is_insert)-->
            <button type="submit" class="btn btn-primary">등록</button>
            <!--/if(is_insert)-->

            <!--@if(is_modify)-->
            <button type="submit" class="btn btn-primary">수정</button>
            <!--/if(is_modify)-->

            <a href="{{entity}}_list.jsp" class="btn btn-secondary">목록</a>
        </div>
    </form>
</div>

{{form_script}}
\`\`\`
```

### 예제 2: Workers API 템플릿

**파일: `.claude/templates/api-endpoint.md`**

```markdown
# API 엔드포인트 템플릿 (Cloudflare Workers)

## 사용 방법
RESTful API 엔드포인트를 생성할 때 이 템플릿을 사용하세요.
- {{Entity}}: PascalCase 엔티티명 (예: User, Product)
- {{entity}}: camelCase 엔티티명 (예: user, product)
- {{table}}: 테이블명 (예: users, products)

## Service 파일: /src/services/{{entity}}Service.js

\`\`\`javascript
export class {{Entity}}Service {
  constructor(env) {
    this.env = env;
  }

  async get{{Entity}}(id) {
    // KV 캐시 확인
    const cacheKey = \`{{entity}}:\${id}\`;
    const cached = await this.env.KV.get(cacheKey, { type: 'json' });
    if (cached) return cached;

    // D1 조회
    const {{entity}} = await this.env.DB
      .prepare('SELECT * FROM {{table}} WHERE id = ?')
      .bind(id)
      .first();

    if (!{{entity}}) {
      const error = new Error('{{Entity}} not found');
      error.name = 'NotFoundError';
      throw error;
    }

    // 캐시 저장 (1시간)
    await this.env.KV.put(cacheKey, JSON.stringify({{entity}}), {
      expirationTtl: 3600
    });

    return {{entity}};
  }

  async list{{Entity}}s(filters = {}) {
    let query = 'SELECT * FROM {{table}} WHERE 1=1';
    const params = [];

    if (filters.status) {
      query += ' AND status = ?';
      params.push(filters.status);
    }

    query += ' ORDER BY created_at DESC LIMIT ?';
    params.push(filters.limit || 50);

    const { results } = await this.env.DB
      .prepare(query)
      .bind(...params)
      .all();

    return results;
  }

  async create{{Entity}}(data) {
    const result = await this.env.DB
      .prepare('INSERT INTO {{table}} (name, status, created_at) VALUES (?, ?, ?)')
      .bind(data.name, data.status || 'active', new Date().toISOString())
      .run();

    const id = result.meta.last_row_id;

    // 캐시 무효화 (선택)
    await this.env.KV.delete(\`{{entity}}:\${id}\`);

    return { id, ...data };
  }

  async update{{Entity}}(id, data) {
    const result = await this.env.DB
      .prepare('UPDATE {{table}} SET name = ?, status = ?, updated_at = ? WHERE id = ?')
      .bind(data.name, data.status, new Date().toISOString(), id)
      .run();

    if (result.meta.changes === 0) {
      const error = new Error('{{Entity}} not found');
      error.name = 'NotFoundError';
      throw error;
    }

    // 캐시 무효화
    await this.env.KV.delete(\`{{entity}}:\${id}\`);

    return { id, ...data };
  }

  async delete{{Entity}}(id) {
    const result = await this.env.DB
      .prepare('DELETE FROM {{table}} WHERE id = ?')
      .bind(id)
      .run();

    if (result.meta.changes === 0) {
      const error = new Error('{{Entity}} not found');
      error.name = 'NotFoundError';
      throw error;
    }

    // 캐시 무효화
    await this.env.KV.delete(\`{{entity}}:\${id}\`);

    return true;
  }
}
\`\`\`

## Route 파일: /src/routes/{{entity}}s.js

\`\`\`javascript
import { Hono } from 'hono';
import { {{Entity}}Service } from '../services/{{entity}}Service.js';

const {{entity}}s = new Hono();

// GET /{{entity}}s - 목록 조회
{{entity}}s.get('/', async (c) => {
  const status = c.req.query('status');
  const limit = parseInt(c.req.query('limit') || '50');

  const service = new {{Entity}}Service(c.env);
  const results = await service.list{{Entity}}s({ status, limit });

  return c.json({ data: results });
});

// GET /{{entity}}s/:id - 상세 조회
{{entity}}s.get('/:id', async (c) => {
  const id = c.req.param('id');

  const service = new {{Entity}}Service(c.env);
  const {{entity}} = await service.get{{Entity}}(id);

  return c.json({ data: {{entity}} });
});

// POST /{{entity}}s - 생성
{{entity}}s.post('/', async (c) => {
  const body = await c.req.json();

  // 간단한 유효성 검증
  if (!body.name) {
    return c.json({ error: 'Name is required' }, 400);
  }

  const service = new {{Entity}}Service(c.env);
  const {{entity}} = await service.create{{Entity}}(body);

  return c.json({ data: {{entity}} }, 201);
});

// PUT /{{entity}}s/:id - 수정
{{entity}}s.put('/:id', async (c) => {
  const id = c.req.param('id');
  const body = await c.req.json();

  const service = new {{Entity}}Service(c.env);
  const {{entity}} = await service.update{{Entity}}(id, body);

  return c.json({ data: {{entity}} });
});

// DELETE /{{entity}}s/:id - 삭제
{{entity}}s.delete('/:id', async (c) => {
  const id = c.req.param('id');

  const service = new {{Entity}}Service(c.env);
  await service.delete{{Entity}}(id);

  return c.json({ message: '{{Entity}} deleted successfully' });
});

export default {{entity}}s;
\`\`\`

## 등록: /src/index.js에 추가

\`\`\`javascript
import {{entity}}sRoutes from './routes/{{entity}}s.js';

app.route('/{{entity}}s', {{entity}}sRoutes);
\`\`\`
```

### 예제 3: Pages ViewLogic 템플릿

**파일: `.claude/templates/pages-viewlogic.md`**

```markdown
# ViewLogic 페이지 템플릿 (Cloudflare Pages)

## 사용 방법
ViewLogic 패턴을 사용한 페이지 생성 템플릿입니다.

## Logic 파일: /src/logic/{{folder}}/{{page}}.js

\`\`\`javascript
export default {
    layout: 'default',

    data() {
        return {
            items: [],
            loading: false,
            selectedItem: null
        }
    },

    async mounted() {
        await this.loadData();
    },

    methods: {
        async loadData() {
            this.loading = true;
            try {
                const response = await this.$api.get('/api/{{resource}}');
                this.items = response.data;
            } catch (error) {
                console.error('Failed to load data:', error);
                alert('데이터 로딩 실패');
            } finally {
                this.loading = false;
            }
        },

        async handleSubmit() {
            const formData = {
                // 폼 데이터 수집
            };

            try {
                await this.$api.post('/api/{{resource}}', formData);
                alert('저장되었습니다.');
                await this.loadData();
            } catch (error) {
                console.error('Failed to save:', error);
                alert('저장 실패');
            }
        },

        viewDetail(id) {
            this.navigateTo('/{{folder}}/detail', { id });
        }
    },

    computed: {
        filteredItems() {
            // 필터링 로직
            return this.items;
        }
    }
}
\`\`\`

## View 파일: /src/views/{{folder}}/{{page}}.html

\`\`\`html
<div class="container py-4">
    <h1 class="mb-4">{{Title}}</h1>

    <!-- 로딩 상태 -->
    <div v-if="loading" class="text-center">
        <div class="spinner-border" role="status">
            <span class="visually-hidden">로딩중...</span>
        </div>
    </div>

    <!-- 데이터 목록 -->
    <div v-else>
        <div class="row">
            <div v-for="item in filteredItems" :key="item.id" class="col-md-4 mb-3">
                <div class="card">
                    <div class="card-body">
                        <h5 class="card-title">{{ item.title }}</h5>
                        <p class="card-text">{{ item.description }}</p>
                        <button @click="viewDetail(item.id)" class="btn btn-primary">
                            상세보기
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
\`\`\`
```

## Claude Code 활용 방법

### 템플릿 사용 프롬프트

```
프롬프트: ".claude/templates/crud-list.md 템플릿을 사용해서
제품(Product) 목록 페이지를 만들어줘. 테이블은 tb_product이고
name, price, category 필드로 검색 가능하게 해줘."
```

**Claude Code의 동작:**
1. `.claude/templates/crud-list.md` 읽기
2. 플레이스홀더 치환:
   - `{{entity}}` → `Product`
   - `{{table}}` → `tb_product`
   - `{{fields}}` → `name,price,category`
   - `{{folder}}` → `product`
3. 템플릿 기반으로 3개 파일 생성:
   - `/product/product_list.jsp`
   - `/html/product/product_list.html`
   - `/src/dao/ProductDao.java`

### 템플릿 없이 요청한 경우와 비교

**템플릿 없이:**
- Claude Code가 자체 판단으로 코드 생성
- 프로젝트 패턴과 다를 수 있음
- 유효성 검증 누락 가능
- 권한 체크 누락 가능

**템플릿 사용:**
- 검증된 패턴을 그대로 사용
- 일관된 구조 유지
- 필수 요소 자동 포함
- 버그 발생 가능성 감소

## 템플릿 관리 팁

### 1. 실제 운영 코드를 템플릿으로

가장 좋은 템플릿은 실제로 작동하는 코드입니다.

### 2. 주석으로 설명 추가

```javascript
// {{Entity}}: 엔티티명을 PascalCase로 입력 (예: User, Product)
export class {{Entity}}Service {
  // ...
}
```

### 3. 플레이스홀더 일관성

- `{{entity}}`: camelCase
- `{{Entity}}`: PascalCase
- `{{ENTITY}}`: UPPER_CASE
- `{{table}}`: snake_case

### 4. 필수 요소 포함

- 유효성 검증
- 에러 처리
- 권한 체크
- 로깅

### 5. 주기적 업데이트

베스트 프랙티스가 변경되면 템플릿도 업데이트하세요.

## 다음 단계

- [.claude/rules 작성 가이드](claude-rules.md)
- [CLAUDE.md 작성 가이드](claude-md.md)
- [MCP 서버 설정 및 활용](mcp-setup.md)

---

[← 목차로 돌아가기](../_sidebar.md)
