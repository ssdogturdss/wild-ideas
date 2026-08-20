---
name: React Query auth gate after login
description: How to prevent AuthGate from bouncing back to /login after successful login due to stale 401 cache
---

# React Query Auth Gate — Post-Login Pattern

## Rule
After a successful login mutation, always invalidate the `useGetMe` query before calling `setLocation("/")`. Otherwise AuthGate sees the stale 401 result and immediately redirects back to `/login`.

Also, AuthGate must check `isFetching` (not just `isLoading`) to show the loading spinner while the refetch is in flight.

## Pattern in login.tsx onSuccess
```typescript
onSuccess: () => {
  queryClient.invalidateQueries({ queryKey: getGetMeQueryKey() });
  setLocation("/");
}
```

## Pattern in AuthGate (App.tsx)
```typescript
const { data: user, isLoading, isFetching, error } = useGetMe();
if (isLoading || isFetching) return <LoadingSpinner />;
if (error || !user) return <Redirect to="/login" />;
```

**Why:** React Query caches failed (401) results. When the user logs in and the session cookie is set, the old cache entry still shows `user=undefined`. Without invalidation + isFetching guard, the redirect fires before the fresh fetch completes.
