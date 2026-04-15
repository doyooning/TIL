
---
# 데이터 개요

- 운영자
- 경기
- 팀
- 선수
- 경기 라인업
- 경기 이벤트
    - 득점
    - 경고/퇴장
    - 교체
- 공지 또는 경기 상태

### 테이블 설계
```sql
create table public.admins (
  id uuid primary key default gen_random_uuid(),
  email text unique not null,
  name text not null,
  created_at timestamptz not null default now()
);

create table public.matches (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  home_team text not null,
  away_team text not null,
  location text,
  match_date timestamptz not null,
  youtube_url text,
  status text not null default 'scheduled', -- scheduled, live, ended
  home_score int not null default 0,
  away_score int not null default 0,
  created_at timestamptz not null default now()
);

create table public.lineups (
  id uuid primary key default gen_random_uuid(),
  match_id uuid not null references public.matches(id) on delete cascade,
  team_side text not null, -- home / away
  player_name text not null,
  jersey_number int,
  is_starter boolean not null default false,
  created_at timestamptz not null default now()
);

create table public.match_events (
  id uuid primary key default gen_random_uuid(),
  match_id uuid not null references public.matches(id) on delete cascade,
  minute int,
  team_side text not null, -- home / away
  event_type text not null, -- goal, yellow, red, sub_in, sub_out
  player_name text,
  description text,
  created_at timestamptz not null default now()
);
```

