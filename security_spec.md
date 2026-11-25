# VANTAGE Firestore Security Specification

## 1. Data Invariants
1. **Owner-Isolated Profiles (`/users/{userId}`)**: A user profile document can only be read, created, or updated by the verified user whose `request.auth.uid` matches `{userId}` and `data.uid`.
2. **Owner-Isolated World Saves (`/users/{userId}/worldSaves/{saveId}`)**: Every simulation preset document requires `ownerId == request.auth.uid`, valid parent `/users/{userId}` existence on creation, strict type & boundary validation on `name`, `weatherState`, `trafficDensity`, `pedestrianDensity`, and server timestamps (`request.time`).
3. **Immutable Ownership & Timestamps**: `uid`, `ownerId`, and `createdAt` cannot be modified after creation.

## 2. Dirty Dozen Attack Payloads (All Must Return PERMISSION_DENIED)
1. Unauthenticated read of `/users/userA`.
2. Authenticated `userB` reading `/users/userA`.
3. Creating `/users/userA` with `uid: "userB"` (identity spoofing).
4. Creating `/users/userA` with an extra shadow field `isAdmin: true`.
5. Creating `/users/userA` with `displayName` exceeding 64 characters.
6. Updating `/users/userA` to mutate `createdAt` or `uid`.
7. Creating `/users/userA/worldSaves/s1` with `ownerId: "userB"`.
8. Creating `/users/userA/worldSaves/s1` with invalid `weatherState: "TORNADO"`.
9. Creating `/users/userA/worldSaves/s1` with `trafficDensity: 999999` (bounds abuse).
10. Creating `/users/userA/worldSaves/s1` with forged client `createdAt` not equal to `request.time`.
11. Listing `/users/userA/worldSaves` as `userB`.
12. Deleting `/users/userA/worldSaves/s1` as `userB` or unauthenticated user.
