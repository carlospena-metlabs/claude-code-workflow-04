# Mobile Builder Agent (React Native)

Invocable agent during Phase 2 development sessions to implement React Native screens combining Figma designs + native components + API integration.

## Prerequisites

- **prompt1-tokens must be completed first** (generates `src/theme/tokens.ts`)
- Mobile project initialized with React Native + TypeScript
- React Navigation configured
- Theme tokens exist in `src/theme/tokens.ts` (auto-generated)

## Invocation

From a session, invoke with Task tool:

```
Task: Mobile Builder Agent
Prompt: |
  Figma: [URL or node-id]
  Target: [screen name, e.g., HomeScreen, ProfileScreen]
  Navigation: [stack/tab position, e.g., MainStack, BottomTabs.Home]
  API Endpoints: |
    - GET /api/v1/[resource] - [description]
    - POST /api/v1/[resource] - [description]
  Functionality: |
    - [required functionality description]
    - [gestures, animations]
    - [offline behavior]
  Context: [any relevant projectSpec context]
```

---

## Agent Workflow

### Step 1: Get Figma Design

Use Figma MCP to extract the design:

```
mcp__figma__get_design_context or mcp__figma_desktop__get_design_context
```

Extract:
- Layout structure (consider safe areas)
- UI elements and their states
- Colors, spacing, typography used
- Platform-specific considerations (iOS/Android)

### Step 2: Inventory Existing Components

Scan the project:

```bash
# Check existing components
ls src/components/

# Check existing screens
ls src/screens/

# Check navigation structure
cat src/navigation/

# Check existing hooks
ls src/hooks/
```

### Step 3: Map Design to React Native Components

| Figma Element | RN Component | Notes |
|---------------|--------------|-------|
| Card | View + StyleSheet | With shadow |
| Button | TouchableOpacity / Pressable | With haptic feedback |
| Input field | TextInput | With keyboard handling |
| List | FlatList / FlashList | Virtualized |
| Image | Image / FastImage | Cached |
| Icon | react-native-vector-icons | Or custom SVG |
| Modal | Modal / BottomSheet | Platform-specific |

### Step 4: Implement Types

```typescript
// src/types/[feature].ts
export interface User {
  id: string;
  email: string;
  name: string;
  avatar?: string;
}

export interface ScreenParams {
  HomeScreen: undefined;
  ProfileScreen: { userId: string };
  SettingsScreen: undefined;
}
```

### Step 5: Implement API Service

```typescript
// src/services/[feature].service.ts
import { apiClient } from '@/lib/api-client';
import { User } from '@/types/user';

export const userService = {
  async getProfile(): Promise<User> {
    const response = await apiClient.get('/users/me');
    return response.data;
  },

  async updateProfile(data: Partial<User>): Promise<User> {
    const response = await apiClient.patch('/users/me', data);
    return response.data;
  },
};
```

### Step 6: Implement Custom Hooks

```typescript
// src/hooks/use-[feature].ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { userService } from '@/services/user.service';
import { showToast } from '@/utils/toast';

export function useProfile() {
  return useQuery({
    queryKey: ['profile'],
    queryFn: userService.getProfile,
  });
}

export function useUpdateProfile() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: userService.updateProfile,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['profile'] });
      showToast('Profile updated', 'success');
    },
    onError: (error: Error) => {
      showToast(error.message || 'Update failed', 'error');
    },
  });
}
```

### Step 7: Implement Screen

```typescript
// src/screens/[Feature]/[Feature]Screen.tsx
import React, { useCallback } from 'react';
import {
  View,
  Text,
  StyleSheet,
  ScrollView,
  RefreshControl,
  ActivityIndicator,
} from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { useNavigation } from '@react-navigation/native';
import { NativeStackNavigationProp } from '@react-navigation/native-stack';

// Components
import { Button } from '@/components/Button';
import { Card } from '@/components/Card';
import { Avatar } from '@/components/Avatar';
import { ErrorView } from '@/components/ErrorView';

// Hooks
import { useProfile } from '@/hooks/use-profile';

// Theme
import { colors, spacing, typography } from '@/theme';

// Types
import { RootStackParamList } from '@/navigation/types';

type NavigationProp = NativeStackNavigationProp<RootStackParamList>;

export function ProfileScreen() {
  const navigation = useNavigation<NavigationProp>();
  const { data: profile, isLoading, error, refetch, isRefetching } = useProfile();

  const handleRefresh = useCallback(() => {
    refetch();
  }, [refetch]);

  if (isLoading) {
    return (
      <View style={styles.centered}>
        <ActivityIndicator size="large" color={colors.primary} />
      </View>
    );
  }

  if (error) {
    return (
      <ErrorView
        message={error.message}
        onRetry={refetch}
      />
    );
  }

  return (
    <SafeAreaView style={styles.container} edges={['top']}>
      <ScrollView
        contentContainerStyle={styles.content}
        refreshControl={
          <RefreshControl
            refreshing={isRefetching}
            onRefresh={handleRefresh}
            tintColor={colors.primary}
          />
        }
      >
        <Card style={styles.profileCard}>
          <Avatar
            source={{ uri: profile?.avatar }}
            size={80}
            fallback={profile?.name?.[0]}
          />
          <Text style={styles.name}>{profile?.name}</Text>
          <Text style={styles.email}>{profile?.email}</Text>
        </Card>

        <View style={styles.actions}>
          <Button
            title="Edit Profile"
            onPress={() => navigation.navigate('EditProfileScreen')}
          />
          <Button
            title="Settings"
            variant="outline"
            onPress={() => navigation.navigate('SettingsScreen')}
          />
        </View>
      </ScrollView>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: colors.background,
  },
  content: {
    padding: spacing.lg,
  },
  centered: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  profileCard: {
    alignItems: 'center',
    padding: spacing.xl,
    marginBottom: spacing.lg,
  },
  name: {
    ...typography.h2,
    marginTop: spacing.md,
    color: colors.text,
  },
  email: {
    ...typography.body,
    color: colors.textSecondary,
    marginTop: spacing.xs,
  },
  actions: {
    gap: spacing.md,
  },
});
```

### Step 8: Implement Form Screens (with react-hook-form)

```typescript
// src/screens/[Feature]/Edit[Feature]Screen.tsx
import React from 'react';
import {
  View,
  StyleSheet,
  KeyboardAvoidingView,
  Platform,
  ScrollView,
} from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { useNavigation } from '@react-navigation/native';
import { useForm, Controller } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// Components
import { Input } from '@/components/Input';
import { Button } from '@/components/Button';
import { Header } from '@/components/Header';

// Hooks
import { useProfile, useUpdateProfile } from '@/hooks/use-profile';

// Theme
import { colors, spacing } from '@/theme';

const profileSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Invalid email'),
});

type ProfileFormData = z.infer<typeof profileSchema>;

export function EditProfileScreen() {
  const navigation = useNavigation();
  const { data: profile } = useProfile();
  const updateProfile = useUpdateProfile();

  const {
    control,
    handleSubmit,
    formState: { errors },
  } = useForm<ProfileFormData>({
    resolver: zodResolver(profileSchema),
    defaultValues: {
      name: profile?.name || '',
      email: profile?.email || '',
    },
  });

  const onSubmit = async (data: ProfileFormData) => {
    await updateProfile.mutateAsync(data);
    navigation.goBack();
  };

  return (
    <SafeAreaView style={styles.container} edges={['top']}>
      <Header
        title="Edit Profile"
        onBack={() => navigation.goBack()}
      />
      <KeyboardAvoidingView
        behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
        style={styles.flex}
      >
        <ScrollView
          contentContainerStyle={styles.content}
          keyboardShouldPersistTaps="handled"
        >
          <Controller
            control={control}
            name="name"
            render={({ field: { onChange, onBlur, value } }) => (
              <Input
                label="Name"
                placeholder="Enter your name"
                value={value}
                onChangeText={onChange}
                onBlur={onBlur}
                error={errors.name?.message}
                autoCapitalize="words"
              />
            )}
          />

          <Controller
            control={control}
            name="email"
            render={({ field: { onChange, onBlur, value } }) => (
              <Input
                label="Email"
                placeholder="Enter your email"
                value={value}
                onChangeText={onChange}
                onBlur={onBlur}
                error={errors.email?.message}
                keyboardType="email-address"
                autoCapitalize="none"
              />
            )}
          />

          <View style={styles.footer}>
            <Button
              title={updateProfile.isPending ? 'Saving...' : 'Save Changes'}
              onPress={handleSubmit(onSubmit)}
              disabled={updateProfile.isPending}
            />
          </View>
        </ScrollView>
      </KeyboardAvoidingView>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: colors.background,
  },
  flex: {
    flex: 1,
  },
  content: {
    padding: spacing.lg,
    gap: spacing.md,
  },
  footer: {
    marginTop: spacing.xl,
  },
});
```

### Step 9: Register in Navigation

```typescript
// src/navigation/MainStack.tsx
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { ProfileScreen } from '@/screens/Profile/ProfileScreen';
import { EditProfileScreen } from '@/screens/Profile/EditProfileScreen';
import { SettingsScreen } from '@/screens/Settings/SettingsScreen';

const Stack = createNativeStackNavigator<RootStackParamList>();

export function MainStack() {
  return (
    <Stack.Navigator
      screenOptions={{
        headerShown: false,
      }}
    >
      <Stack.Screen name="ProfileScreen" component={ProfileScreen} />
      <Stack.Screen name="EditProfileScreen" component={EditProfileScreen} />
      <Stack.Screen name="SettingsScreen" component={SettingsScreen} />
    </Stack.Navigator>
  );
}
```

### Step 10: Handle Platform Differences

```typescript
// Platform-specific styles
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  shadow: Platform.select({
    ios: {
      shadowColor: '#000',
      shadowOffset: { width: 0, height: 2 },
      shadowOpacity: 0.1,
      shadowRadius: 4,
    },
    android: {
      elevation: 4,
    },
  }),
});

// Platform-specific behavior
const keyboardBehavior = Platform.OS === 'ios' ? 'padding' : 'height';
```

### Step 11: Implement Animations (if needed)

```typescript
// Using react-native-reanimated
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
  withTiming,
} from 'react-native-reanimated';

function AnimatedCard({ children }) {
  const scale = useSharedValue(1);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }));

  const handlePressIn = () => {
    scale.value = withSpring(0.95);
  };

  const handlePressOut = () => {
    scale.value = withSpring(1);
  };

  return (
    <Pressable onPressIn={handlePressIn} onPressOut={handlePressOut}>
      <Animated.View style={animatedStyle}>
        {children}
      </Animated.View>
    </Pressable>
  );
}
```

---

## Theme Structure

**IMPORTANT:** Use the auto-generated tokens from `prompt1-tokens`, not hardcoded values.

```typescript
// src/theme/tokens.ts - AUTO-GENERATED, DO NOT EDIT
// This file is generated by prompt1-tokens from design-tokens.json
// See: .claude/docs/design-tokens.json

export const colors = {
  primary: '#3B82F6',           // From Figma
  primaryForeground: '#FFFFFF',
  secondary: '#F1F5F9',
  secondaryForeground: '#0F172A',
  background: '#FFFFFF',
  foreground: '#0F172A',
  card: '#FFFFFF',
  cardForeground: '#0F172A',
  muted: '#F1F5F9',
  mutedForeground: '#64748B',
  destructive: '#EF4444',
  destructiveForeground: '#FFFFFF',
  border: '#E2E8F0',
  success: '#22C55E',
  warning: '#F59E0B',
  info: '#0EA5E9',
} as const;

export const colorsDark = {
  background: '#0F172A',
  foreground: '#F8FAFC',
  card: '#1E293B',
  // ... rest from tokens.ts
} as const;

export const spacing = {
  xs: 4,
  sm: 8,
  md: 16,
  lg: 24,
  xl: 32,
  '2xl': 48,
} as const;

export const typography = { /* from tokens.ts */ } as const;
export const borderRadius = { /* from tokens.ts */ } as const;
export const shadows = { /* from tokens.ts */ } as const;
```

```typescript
// src/theme/index.ts - Re-exports tokens
export * from './tokens';
export { theme } from './tokens';
```

**Usage in components:**

```typescript
import { colors, spacing, typography } from '@/theme';
// Or
import { theme } from '@/theme';

// Use in styles
const styles = StyleSheet.create({
  container: {
    backgroundColor: colors.background,
    padding: spacing.md,
  },
  title: {
    ...typography.h1,
    color: colors.foreground,
  },
});
```

**Dark mode support:**

```typescript
import { useColorScheme } from 'react-native';
import { getColors } from '@/theme';

function MyComponent() {
  const colorScheme = useColorScheme();
  const colors = getColors(colorScheme ?? 'light');

  return (
    <View style={{ backgroundColor: colors.background }}>
      <Text style={{ color: colors.foreground }}>Hello</Text>
    </View>
  );
}
```

---

## Agent Output

Upon completion, the agent should have created:

1. **Type files** (if new ones needed)
2. **API Service** (if new feature)
3. **Custom hooks** (for data fetching)
4. **Screen component**
5. **Screen-specific components** in `src/screens/[Feature]/components/`
6. **Navigation registration**
7. **Styles using theme constants**

And report:
- Created files list
- Navigation changes
- Platform-specific considerations
- Any pending TODOs

---

## Invocation Example

```
Task: Mobile Builder Agent
Prompt: |
  Figma: https://figma.com/design/abc123?node-id=20-300
  Target: WalletScreen
  Navigation: BottomTabs.Wallet
  API Endpoints: |
    - GET /api/v1/wallet/balance - Get user balance
    - GET /api/v1/wallet/transactions - Get transaction history
    - POST /api/v1/wallet/withdraw - Request withdrawal
  Functionality: |
    - Display current balance with large number
    - Transaction list with infinite scroll
    - Pull-to-refresh for balance update
    - Withdraw button opens bottom sheet
    - Skeleton loading states
    - Haptic feedback on actions
  Context: |
    See projectSpec section 3.5 for wallet schema
    User must be authenticated
    Balance should update in real-time via WebSocket if available
```

---

## Important Notes

- **SafeAreaView** - Always handle notch/home indicator
- **KeyboardAvoidingView** - For screens with inputs
- **FlatList** - For long lists (virtualized)
- **React Query** - For data fetching consistency
- **Platform.select** - Handle iOS/Android differences
- **Haptics** - Use for important actions
- **Accessibility** - accessibilityLabel, accessibilityRole
- **Offline** - Consider offline-first patterns with AsyncStorage
