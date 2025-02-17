=============
Custom Events
=============

.. javadoc-import::
    org.spongepowered.api.entity.living.player.server.ServerPlayer
    org.spongepowered.api.event.Cancellable
    org.spongepowered.api.event.Event
    org.spongepowered.api.event.cause.Cause
    org.spongepowered.api.event.entity.AffectEntityEvent
    org.spongepowered.api.event.impl.AbstractEvent

You can write your own event classes and dispatch those events using the method described above. An event class must
extend the :javadoc:`AbstractEvent` class, thus implementing the :javadoc:`Event` interface. Depending on the exact
nature of the event, more interfaces should be implemented, like :javadoc:`Cancellable` for events that can be
cancelled by a listener or interfaces like :javadoc:`AffectEntityEvent` clarifying what sort of object is affected by
your event.

Example: Custom Event Class
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following class describes an event indicating a :javadoc:`ServerPlayer` has come in contact with FLARD and is now about to
mutate in a way specified by the event. Since the event can be cancelled by listeners, it implements the ``Cancellable``
interface.

Since generally custom events are intended to be listened to by other plugins, it is in your best interest to document
them appropriately. This includes a list of objects typically found in the :javadoc:`Cause`. In the below example, it
would probably be mentioned that the root cause is generally an object of the fictitious ``FLARDSource`` class.

.. code-block:: java

    import org.spongepowered.api.entity.living.player.server.ServerPlayer;
    import org.spongepowered.api.event.Cancellable;
    import org.spongepowered.api.event.cause.Cause;
    import org.spongepowered.api.event.impl.AbstractEvent;

    public class PlayerMutationEvent extends AbstractEvent implements Cancellable {

        public enum Mutation {
            COMPULSIVE_POETRY,
            ROTTED_SOCKS,
            SPONTANEOUS_COMBUSTION;
        };

        private final Cause cause;
        private final ServerPlayer victim;
        private final Mutation mutation;
        private boolean cancelled = false;

        public PlayerMutationEvent(ServerPlayer victim, Mutation mutation, Cause cause) {
            this.victim = victim;
            this.mutation = mutation;
            this.cause = cause;
        }

        public ServerPlayer victim() {
            return this.victim;
        }

        public Mutation mutation() {
            return this.mutation;
        }

        @Override
        public boolean isCancelled() {
            return this.cancelled;
        }

        @Override
        public void setCancelled(boolean cancel) {
            this.cancelled = cancel;
        }

        @Override
        public Cause cause() {
            return this.cause;
        }

    }

Example: Fire Custom Event
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: java

    import org.spongepowered.api.event.cause.Cause;
    import org.spongepowered.api.event.cause.EventContext;
    import org.spongepowered.api.event.cause.EventContextKeys;
    import org.spongepowered.api.Sponge;

    PluginContainer plugin = ...;
    EventContext eventContext = EventContext.builder().add(EventContextKeys.PLUGIN, plugin).build();

    PlayerMutationEvent event = new PlayerMutationEvent(victim, PlayerMutationEvent.Mutation.ROTTED_SOCKS,
            Cause.of(eventContext, plugin));
    Sponge.eventManager().post(event);
    if (!event.isCancelled()) {
        // Mutation code
    }

Bear in mind that you need to supply a non-empty cause. If your event was ``Cancellable``, make sure that it was not
cancelled before performing the action described by the event.

Example: Listen for Custom Event
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: java

    import net.kyori.adventure.text.Component;
    import org.spongepowered.api.event.Listener;

    @Listener
    public void onPrivateMessage(PlayerMutationEvent event) {
        if (event.mutation() == PlayerMutationEvent.Mutation.SPONTANEOUS_COMBUSTION) {
            event.setCancelled(true);
            event.victim().sendMessage(Component.text("You cannot combust here, this is a non-smoking area!"));
        }
    }
